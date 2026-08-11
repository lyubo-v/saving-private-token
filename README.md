# saving-private-tokens

A Claude Code skill that instructs Claude to minimize token and computation spend on every task.

## Install

```bash
gh release download --repo LyuboVoynov/saving-private-tokens --pattern "*.skill" \
  && unzip -o ai-cost-optimization.skill -d ~/.claude/skills \
  && rm ai-cost-optimization.skill
```

No releases yet? Install directly from main:

```bash
curl -L https://github.com/LyuboVoynov/saving-private-tokens/raw/main/ai-cost-optimization.skill -o /tmp/skill.zip \
  && unzip -o /tmp/skill.zip -d ~/.claude/skills \
  && rm /tmp/skill.zip
```

Then invoke in any Claude Code session:

```
/ai-cost-optimization
```

## What it does

When loaded, the skill directs Claude to:

- **Classify task complexity** (Level 0–3) before choosing effort level
- **Allocate reasoning proportionally** — spend computation where it changes the answer, stop when confidence is sufficient
- **Filter and prioritize context** — ignore irrelevant content, deduplicate, never prune meaning-critical tokens (negations, numbers, conditions)
- **Budget extended thinking** — enable only when the task warrants it, always cap the token budget
- **Minimize output** — generate the shortest complete answer; no restatements, filler, or unrequested background
- **Use tools sparingly** — only when the task can't be solved without them
- **Route to the minimum capable model tier** — escalate only on demonstrated failure, de-escalate mid-task when complexity drops

## Files

```
ai-cost-optimization.skill   ← zip archive, unpack to ~/.claude/skills/
├── SKILL.md                 ← operating instructions loaded by Claude
└── references/
    ├── pricing-snapshot.md  ← model capability ladder for routing decisions
    └── cost-formulas.md     ← break-even math for escalation and caching
```
