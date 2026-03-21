# Self-Evolution Protocol

The /blog-post skill is designed to improve with every use. Each content package produced feeds back into the skill's knowledge, templates, and strategies.

## Evolution Substrate

### What Gets Better Over Time

| Layer | How It Evolves | Storage |
|-------|---------------|---------|
| **Hook library** | Track which hooks achieve 80%+ 3-second retention. Promote to template. | `references/proven-hooks.md` |
| **Veo prompts** | Prompts that produce usable clips get saved. Duds get annotated with failure reason. | `references/veo-prompt-library.md` |
| **Platform patterns** | Which post formats drive DM sends (strongest algorithm signal). | `references/platform-adaptation.md` |
| **Distribution timing** | Optimal posting times refined per audience segment. | `strategy/` templates |
| **Content pillars** | Recurring themes that perform well. | `references/content-pillars.md` |
| **Quality gates** | New checklist items from post-mortems of underperforming content. | `references/quality-checklist.md` |

### Feedback Loop Architecture

```
PUBLISH → MEASURE → EXTRACT PATTERNS → UPDATE SKILL → NEXT PUBLISH
```

1. **Publish** content package to platforms
2. **Measure** after 48 hours: DM sends, saves, watch completion, reach
3. **Extract patterns**: What worked? What failed? Why?
4. **Update skill**: Promote winning patterns to templates, demote losers
5. **Next publish** benefits from accumulated knowledge

### What to Measure (Priority Order)

| Metric | Why | Platform |
|--------|-----|----------|
| DM sends / reach | Strongest algorithm signal for new audience reach | Instagram |
| 3-second retention | Determines if hook works | Instagram Reels |
| Save rate | Indicates lasting value | Instagram, LinkedIn |
| Thread completion | Determines if narrative holds | X |
| Watch-through rate | Determines if pacing works | Reels, video |
| Click-through to blog | Measures CTA effectiveness | All |
| Time on page | Measures content quality | broomva.tech |

### A/B Testing Protocol

For each content package, vary ONE element across platforms:
- Same content, different hooks (test hook formulas)
- Same hook, different posting times (test timing)
- Same content, different media (image vs. carousel vs. reel)
- Same message, different tone (technical vs. conversational)

Track which variation wins. After 5+ data points per variable, promote the winner to default.

## Self-Improvement Triggers

### After Every Publish

The agent should ask:
1. "Which platform performed best? What was different about that adaptation?"
2. "Did any hook significantly outperform others? Save it to proven-hooks."
3. "Did any Veo prompt produce an unusable clip? Annotate why."
4. "Were there quality gate failures in production? Add preventive checks."

### Monthly Review

1. Review all content packages from the past month
2. Rank by engagement per platform
3. Identify the top 3 patterns and bottom 3 patterns
4. Update skill references with findings
5. Prune strategies that consistently underperform
6. Add new strategies observed from competitors or trends

### Quarterly Evolution

1. Research current algorithm changes (Instagram, X, LinkedIn)
2. Update platform-adaptation.md with new best practices
3. Audit compounding skills — are there new skills worth adding?
4. Review media tooling — new AI models, new capabilities?
5. Update reel-production.md with new techniques
6. Version bump the skill and push to GitHub

## Content Pillars (Bootstrap)

Define 3-5 recurring themes. Each content package should fit a pillar:

| Pillar | Description | Audience |
|--------|-------------|----------|
| **Build Logs** | What we built, how, and what happened | Developers |
| **Agent Architecture** | How the Agent OS and skill stack work | AI builders |
| **Meta-Content** | Content about creating content (this post) | Creators |
| **Open Source** | What we released and why it matters | Community |
| **Contrarian Takes** | Challenge conventional wisdom with evidence | Broad |

Each pillar has its own optimal format:
- Build logs → X thread + blog post
- Agent architecture → blog post + LinkedIn
- Meta-content → all platforms (universal appeal)
- Open source → X thread + blog post
- Contrarian takes → X post + LinkedIn

## Compounding Skills Ecosystem

The /blog-post skill compounds on a growing ecosystem. New skills should be evaluated and integrated when they provide capabilities the skill currently handles manually.

### Currently Compounding

| Skill | Role |
|-------|------|
| `/content-creation` | Storytelling, social patterns, AI assets |
| `/deep-research` | Multi-source research |
| `/pencil` | Carousel design, social cards |
| `/remotion-best-practices` | Video composition |
| `/arcan-glass` | Brand styling |
| `/google-veo` | Veo video generation prompting |
| `/subtitle-generation` | Subtitle/caption generation |

### Candidates for Future Integration

| Skill | What It Would Add | When to Add |
|-------|-------------------|-------------|
| Video editing agent | Automated post-production | When reel volume > 5/week |
| Instagram strategist | Algorithm-aware content optimization | When IG becomes primary channel |
| Content strategy | Analytics-driven pillar management | When running A/B tests |
| OpusClip integration | Long-form to short-form extraction | When producing podcast/long video |

### Integration Criteria

Add a compounding skill when:
1. The skill handles a task the /blog-post skill currently does manually
2. The skill has > 100 installs (validated by community)
3. The skill's output can be consumed by the pipeline without manual intervention
4. Adding it doesn't increase SKILL.md beyond 600 lines (use references for depth)
