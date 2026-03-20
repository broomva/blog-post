# Publishing Automation

## Overview

Distribution uses native CLI tools and REST APIs — no third-party services. Each platform connector is independent; the skill gracefully degrades when a connector is unavailable.

## X/Twitter via xurl

### Setup (one-time)
```bash
# 1. Register app (needs X Developer Portal credentials)
xurl auth apps add broomva --client-id YOUR_CLIENT_ID --client-secret YOUR_CLIENT_SECRET

# 2. Set as default
xurl auth default broomva

# 3. OAuth2 flow (opens browser)
xurl auth oauth2

# 4. Verify
xurl whoami
```

### Posting a Single Tweet
```bash
# Read the post content (first non-header, non-metadata line)
POST_TEXT=$(sed -n '/^## Post/,/^## /{ /^## /d; /^$/d; p; }' x-post.md | head -1)

# Post with optional image
if [ -f media/thumbnails/x-card.png ]; then
    xurl post "$POST_TEXT" --media media/thumbnails/x-card.png
else
    xurl post "$POST_TEXT"
fi
```

### Posting a Thread

Threads are reply chains. The first tweet is a standalone post; subsequent tweets reply to the previous one.

**Parsing x-thread.md**:
The file uses `### N/N` headers to delimit tweets. Lines starting with `📸 Image:` indicate media attachments.

```bash
#!/bin/bash
# publish-thread.sh — Parse x-thread.md and post as thread
THREAD_FILE="$1"
MEDIA_DIR="$(dirname "$THREAD_FILE")/media"
PREV_ID=""

# Extract tweets between ### headers
awk '/^### [0-9]+\/[0-9]+/{if(tweet)print tweet; tweet=""; next} {tweet=tweet" "$0} END{if(tweet)print tweet}' "$THREAD_FILE" | while IFS= read -r tweet_text; do
    # Clean up whitespace
    tweet_text=$(echo "$tweet_text" | sed 's/^ *//;s/ *$//' | tr -s ' ')

    # Check for image reference
    IMAGE=""
    if echo "$tweet_text" | grep -q "📸 Image:"; then
        IMAGE_REF=$(echo "$tweet_text" | grep -o "📸 Image: .*" | sed 's/📸 Image: //')
        # Remove image line from tweet text
        tweet_text=$(echo "$tweet_text" | grep -v "📸 Image:")
        # Resolve image path
        if [ -f "$MEDIA_DIR/png/$IMAGE_REF" ]; then
            IMAGE="$MEDIA_DIR/png/$IMAGE_REF"
        elif [ -f "$IMAGE_REF" ]; then
            IMAGE="$IMAGE_REF"
        fi
    fi

    # Post or reply
    if [ -z "$PREV_ID" ]; then
        # First tweet
        if [ -n "$IMAGE" ]; then
            RESULT=$(xurl post "$tweet_text" --media "$IMAGE" 2>&1)
        else
            RESULT=$(xurl post "$tweet_text" 2>&1)
        fi
    else
        # Reply to previous
        if [ -n "$IMAGE" ]; then
            RESULT=$(xurl reply "$PREV_ID" "$tweet_text" --media "$IMAGE" 2>&1)
        else
            RESULT=$(xurl reply "$PREV_ID" "$tweet_text" 2>&1)
        fi
    fi

    # Extract tweet ID from response
    PREV_ID=$(echo "$RESULT" | jq -r '.data.id // empty' 2>/dev/null)
    if [ -z "$PREV_ID" ]; then
        echo "ERROR: Failed to post tweet. Response: $RESULT"
        exit 1
    fi
    echo "Posted tweet $PREV_ID"
done
```

### xurl Command Reference

| Command | Usage |
|---------|-------|
| `xurl post "text"` | Post a tweet |
| `xurl post "text" --media file.png` | Post with image |
| `xurl reply ID "text"` | Reply to a tweet |
| `xurl read ID` | Read a tweet |
| `xurl search "query" -n 20` | Search posts |
| `xurl whoami` | Check auth status |
| `xurl media upload file.mp4` | Upload media (video/image) |
| `xurl like ID` | Like a post |
| `xurl repost ID` | Repost/retweet |
| `xurl delete ID` | Delete a post |

## LinkedIn via REST API

### Setup (one-time)
1. Create app at [linkedin.com/developers](https://linkedin.com/developers)
2. Add product: "Share on LinkedIn" → grants `w_member_social` scope
3. OAuth2 flow:

```bash
# 1. Get authorization code (open in browser)
CLIENT_ID="your_client_id"
REDIRECT_URI="http://localhost:8080/callback"
open "https://www.linkedin.com/oauth/v2/authorization?response_type=code&client_id=$CLIENT_ID&redirect_uri=$REDIRECT_URI&scope=openid%20profile%20w_member_social"

# 2. Exchange code for token (after browser redirect)
CODE="paste_code_from_redirect_url"
CLIENT_SECRET="your_client_secret"
curl -s -X POST "https://www.linkedin.com/oauth/v2/accessToken" \
  -d "grant_type=authorization_code&code=$CODE&redirect_uri=$REDIRECT_URI&client_id=$CLIENT_ID&client_secret=$CLIENT_SECRET" \
  | jq -r '.access_token' > ~/.config/blog-post/linkedin-token

# 3. Get your member URN
curl -s -H "Authorization: Bearer $(cat ~/.config/blog-post/linkedin-token)" \
  "https://api.linkedin.com/v2/userinfo" \
  | jq -r '.sub' > ~/.config/blog-post/linkedin-urn
```

### Posting
```bash
TOKEN=$(cat ~/.config/blog-post/linkedin-token)
URN=$(cat ~/.config/blog-post/linkedin-urn)

# Extract post body (skip markdown headers and metadata sections)
POST_BODY=$(sed -n '/^## Post$/,/^## Post Metadata$/{ /^## /d; p; }' linkedin-post.md | sed '/^$/d')

curl -s -X POST "https://api.linkedin.com/v2/ugcPosts" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d "{
    \"author\": \"urn:li:person:$URN\",
    \"lifecycleState\": \"PUBLISHED\",
    \"specificContent\": {
      \"com.linkedin.ugc.ShareContent\": {
        \"shareCommentary\": { \"text\": $(echo "$POST_BODY" | jq -Rs .) },
        \"shareMediaCategory\": \"NONE\"
      }
    },
    \"visibility\": { \"com.linkedin.ugc.MemberNetworkVisibility\": \"PUBLIC\" }
  }"
```

### Posting with Image
```bash
# 1. Register upload
UPLOAD_RESPONSE=$(curl -s -X POST "https://api.linkedin.com/v2/assets?action=registerUpload" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d "{
    \"registerUploadRequest\": {
      \"recipes\": [\"urn:li:digitalmediaRecipe:feedshare-image\"],
      \"owner\": \"urn:li:person:$URN\",
      \"serviceRelationships\": [{
        \"relationshipType\": \"OWNER\",
        \"identifier\": \"urn:li:userGeneratedContent\"
      }]
    }
  }")

UPLOAD_URL=$(echo "$UPLOAD_RESPONSE" | jq -r '.value.uploadMechanism["com.linkedin.digitalmedia.uploading.MediaUploadHttpRequest"].uploadUrl')
ASSET_URN=$(echo "$UPLOAD_RESPONSE" | jq -r '.value.asset')

# 2. Upload image
curl -s -X PUT "$UPLOAD_URL" \
  -H "Authorization: Bearer $TOKEN" \
  --upload-file media/thumbnails/linkedin-card.png

# 3. Create post with image
curl -s -X POST "https://api.linkedin.com/v2/ugcPosts" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d "{
    \"author\": \"urn:li:person:$URN\",
    \"lifecycleState\": \"PUBLISHED\",
    \"specificContent\": {
      \"com.linkedin.ugc.ShareContent\": {
        \"shareCommentary\": { \"text\": $(echo "$POST_BODY" | jq -Rs .) },
        \"shareMediaCategory\": \"IMAGE\",
        \"media\": [{
          \"status\": \"READY\",
          \"media\": \"$ASSET_URN\"
        }]
      }
    },
    \"visibility\": { \"com.linkedin.ugc.MemberNetworkVisibility\": \"PUBLIC\" }
  }"
```

## Instagram via Meta Graph API

### Setup (one-time)
1. Convert Instagram to Business/Creator account
2. Link to a Facebook Page
3. Create Meta app at [developers.facebook.com](https://developers.facebook.com)
4. Add Instagram Graph API product
5. Get long-lived access token:

```bash
# Short-lived → long-lived token exchange
SHORT_TOKEN="your_short_lived_token"
APP_SECRET="your_app_secret"
curl -s "https://graph.facebook.com/v19.0/oauth/access_token?grant_type=fb_exchange_token&client_id=$APP_ID&client_secret=$APP_SECRET&fb_exchange_token=$SHORT_TOKEN" \
  | jq -r '.access_token' > ~/.config/blog-post/instagram-token

# Get IG user ID
curl -s "https://graph.facebook.com/v19.0/me/accounts?access_token=$(cat ~/.config/blog-post/instagram-token)" \
  | jq -r '.data[0].id' > /tmp/page_id
curl -s "https://graph.facebook.com/v19.0/$(cat /tmp/page_id)?fields=instagram_business_account&access_token=$(cat ~/.config/blog-post/instagram-token)" \
  | jq -r '.instagram_business_account.id' > ~/.config/blog-post/instagram-user-id
```

### Posting (image must be publicly accessible URL)
```bash
IG_TOKEN=$(cat ~/.config/blog-post/instagram-token)
IG_USER=$(cat ~/.config/blog-post/instagram-user-id)
IMAGE_URL="https://broomva.tech/images/writing/{slug}/hero.png"
CAPTION=$(sed -n '/^## Caption$/,/^## /{ /^## /d; p; }' instagram-post.md)

# Create container → publish
CONTAINER=$(curl -s -X POST "https://graph.facebook.com/v19.0/$IG_USER/media" \
  -d "image_url=$IMAGE_URL" \
  -d "caption=$(echo "$CAPTION" | jq -Rs .)" \
  -d "access_token=$IG_TOKEN" | jq -r '.id')

curl -s -X POST "https://graph.facebook.com/v19.0/$IG_USER/media_publish" \
  -d "creation_id=$CONTAINER" \
  -d "access_token=$IG_TOKEN"
```

## Connector Status Check

Before publishing, verify which platforms are available:

```bash
# X — check xurl auth
xurl whoami >/dev/null 2>&1 && echo "✅ X: ready" || echo "❌ X: run 'xurl auth oauth2'"

# LinkedIn — check token file
[ -f ~/.config/blog-post/linkedin-token ] && echo "✅ LinkedIn: ready" || echo "❌ LinkedIn: setup needed"

# Instagram — check token file
[ -f ~/.config/blog-post/instagram-token ] && echo "✅ Instagram: ready" || echo "❌ Instagram: setup needed"

# broomva.tech — always available
echo "✅ broomva.tech: ready (git)"
```

## Credential Security

- All tokens stored in `~/.config/blog-post/` — never in the repo
- Conversation bridge redacts `--client-secret`, `--client-id`, bearer tokens, and high-entropy strings
- Never log full API responses containing tokens
- LinkedIn tokens expire after 60 days — refresh via `curl` with refresh_token
- Instagram long-lived tokens last 60 days — renew before expiry
