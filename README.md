# saving-private-token

A Claude Code skill that instructs Claude to minimize token and computation spend on every task.

## Install

```bash
gh repo clone LyuboVoynov/saving-private-tokens /tmp/spt -- --depth=1 \
  && cp -r /tmp/spt/ai-cost-optimization ~/.claude/skills/ \
  && rm -rf /tmp/spt
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
ai-cost-optimization/
├── SKILL.md                 ← operating instructions loaded by Claude
└── references/
    ├── pricing-snapshot.md  ← model capability ladder for routing decisions
    └── cost-formulas.md     ← break-even math for escalation and caching
```
