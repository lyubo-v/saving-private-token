# saving-private-tokens

A Claude Code skill that instructs Claude to minimize token and computation spend on every task.

## What it does

When loaded, the skill directs Claude to:

- **Classify task complexity** (Level 0–3) before choosing effort level
- **Allocate reasoning proportionally** — spend computation where it changes the answer, stop when confidence is sufficient
- **Filter and prioritize context** — ignore irrelevant content, deduplicate, never prune meaning-critical tokens (negations, numbers, conditions)
- **Budget extended thinking** — enable only when the task warrants it, always cap the token budget
- **Minimize output** — generate the shortest complete answer; no restatements, filler, or unrequested background
- **Use tools sparingly** — only when the task can't be solved without them
- **Route to the minimum capable model tier** — escalate only on demonstrated failure, de-escalate mid-task when complexity drops

## Usage

Install as a Claude Code skill. Trigger: `ai-cost-optimization`.

## Files

- `ai-cost-optimization.skill` — the packaged skill (zip archive containing `SKILL.md` and references)
  - `SKILL.md` — operating instructions for Claude
  - `references/pricing-snapshot.md` — model capability ladder for routing orientation
  - `references/cost-formulas.md` — break-even math for escalation and caching decisions
