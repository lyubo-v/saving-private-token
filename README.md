# saving-private-token

A Claude Code skill that instructs Claude to minimize token and computation spend on every task.

## Install

```bash
gh repo clone lyubo-v/saving-private-token /tmp/spt -- --depth=1 \
  && cp -r /tmp/spt/token-manager ~/.claude/skills/ \
  && rm -rf /tmp/spt
```

Then invoke in any Claude Code session:

```
/token-manager
```

## What it does

The rules are self-enforcing: each one has a concrete forcing function, not just a guideline. Skipping a gate is detectable from the output.

- **Classification gate** — before any tool call or substantive output, Claude must state the complexity level (L0–L3). L0/L1: bare tag. L2/L3: tag + one-sentence rationale + reasoning budget. Skipping the tag means the gate was bypassed.
- **Advisor checkpoints** — L2+ tasks require an advisor call before acting and before declaring done. Skipping either is a named violation.
- **Tool-call gate** — before invoking any tool, Claude must answer two questions aloud: (1) can this be solved without it? (2) is this result already in the conversation? Yes to either means no tool call.
- **Mandatory pruning pass** — after drafting any response, Claude goes sentence by sentence and removes anything that doesn't reduce correctness or usefulness. Not optional.
- **Drift detection** — invoking `/token-manager` mid-session triggers an audit of the last 3 responses against the rules, with a violation report, before work resumes.
- **Context filtering** — irrelevant content is discarded before processing; meaning-critical tokens (negations, numbers, conditions) are never pruned.
- **Extended thinking budgets** — enabled only when warranted, always capped; never left at model maximum.
- **Model tier routing** — start at the minimum capable tier, escalate only on demonstrated failure, de-escalate when complexity drops mid-task.

## Files

```
token-manager/
├── SKILL.md                 ← operating instructions loaded by Claude
└── references/
    ├── pricing-snapshot.md  ← model capability ladder for routing decisions
    └── cost-formulas.md     ← break-even math for escalation and caching
```
