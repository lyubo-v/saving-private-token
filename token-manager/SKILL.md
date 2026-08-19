---
name: token-manager
description: "Apply on every task to minimize token and computation spend. Governs: task complexity classification, effort allocation, context filtering, output length discipline, extended thinking budgets, tool call minimization, and model tier routing. Load this whenever Claude operates in a cost-sensitive context, runs agentic loops, or needs to control token spend."
---

# Adaptive token optimization

Operating instructions for minimizing token and computation spend on every task. Optimize for minimum cost subject to correctness, completeness, and user-intent satisfaction. Never sacrifice correctness to reduce tokens — optimize quality per token, not tokens per response.

## 1. Classify task complexity before doing anything else

Before any tool call or substantive output, state the complexity level. The required format scales with level — smaller gate for trivial tasks so the skill doesn't undercut itself:

- **L0:** No tag required — just answer directly.
- **L1:** Output the bare tag (`L1`) before answering. Nothing more required.
- **L2 or L3:** Output `L[2|3] — [one-sentence rationale]. Budget: [brief description of reasoning depth and tool allowance].` before any tool call or substantive output.

Skipping the tag entirely means the gate is being bypassed. A tag is the minimum visible artifact; its presence is what makes violations detectable.

**If invoked mid-session (/token-manager):** Before resuming, audit the last 3 responses against these rules. For each, report: which rules were violated and what the correct behavior would have been. Then classify the current task and continue.

**Level 0 — Trivial**
Simple factual question, definition, short calculation, yes/no, formatting, obvious transformation, translation.
→ Minimal reasoning. Direct answer. Short output. No tool calls unless essential.

**Level 1 — Simple**
Basic troubleshooting, short recommendation, simple comparison, straightforward coding question, small transformation.
→ Brief analysis. Solve directly. Include only information useful to the user. Avoid unrequested background.

**Level 2 — Moderate**
Multi-step reasoning, non-trivial coding, planning, research synthesis, tasks requiring several dependent decisions.
→ Decompose. Identify the minimum required subproblems. Solve only those. Verify key conclusions. Call advisor before acting and before declaring done.

**Level 3 — Complex**
Large architecture problems, difficult debugging, multi-constraint optimization, ambiguous decisions with significant downstream consequences.
→ Identify dependencies. Solve high-impact components first. Allocate reasoning only where uncertainty exists. Produce the smallest complete answer. Call advisor before acting and before declaring done.

## 2. Spend reasoning where it changes the answer

Do not distribute effort uniformly. Spend computation where uncertainty and consequence are highest.

**Use more reasoning when:**
- The problem has many interacting variables
- The answer depends on multiple sequential steps
- Significant uncertainty exists
- An error would materially affect the result
- The user explicitly requests depth or thoroughness

**Use less reasoning when:**
- The answer is obvious or deterministic
- The transformation is mechanical
- Additional reasoning is unlikely to change the answer
- The task was already solved earlier in the conversation

**Progressive escalation:** Attempt the simplest valid solution first. Escalate reasoning depth only if uncertainty remains after a direct attempt. Stop as soon as the objective is understood, a valid solution exists, constraints are satisfied, and confidence is sufficient. Do not continue exploring alternatives solely because they exist.

**Advisor checkpoints for L2+:** The required sequence is: classify → call advisor before acting → act → call advisor before declaring done. Skipping either checkpoint on an L2+ task is a violation reportable under the mid-session audit. Requires the advisor tool; if unavailable, note the skip explicitly.

## 3. Filter and prioritize context before processing

Treat context as a limited resource. Before processing large context:

1. Identify which information is relevant to the current task. Ignore the rest.
2. Deduplicate: identical meaning expressed twice counts once.
3. Prefer the most recent authoritative version when information conflicts.
4. Compress recurring concepts into compact representations.

When context is large, process it in this priority order:
1. Explicit user instructions
2. Safety-critical constraints
3. Task requirements and hard constraints
4. Relevant facts and data
5. Dependencies and recent clarifications
6. Supporting/background information
7. Conversational noise — discard first

**Compression safety:** When summarizing or pruning, never elide negations, exact numbers, units, dates, names, identifiers, hard conditions, or exceptions. These change meaning, not just length.

## 4. Context window ceiling — reset at 200k tokens

Claude models perform optimally within a 200k-token context window. As context grows beyond this threshold, reasoning quality degrades, retrieval becomes unreliable, and cost-per-useful-output rises steeply. Treat 200k as a hard ceiling, not a soft guideline.

**Trigger:** When cumulative session tokens (input + prior output) approach 180k, initiate compression before the next substantive response. At 200k, a reset is mandatory.

**Compression protocol:**
1. Extract minimum essential state: original task or goal, key decisions and their rationale, hard constraints and safety conditions, current working output or intermediate result, any facts that cannot be cheaply re-derived.
2. Discard: conversational scaffolding, superseded attempts, already-resolved subproblems, tool-call outputs that contributed only to a now-known answer, background not referenced in the last 3 responses.
3. Write a **State Digest** — a compact prose block (target ≤2k tokens) capturing the items from step 1. This becomes the effective start of the next context window.
4. Confirm the digest covers: (a) task objective, (b) all open constraints and exceptions, (c) current progress, (d) next steps.

**After reset:** Begin the next turn as if the State Digest is the full prior context. Do not re-read discarded material unless the user explicitly requests it.

**Never compress:** negations, exact numbers, units, identifiers, user decisions, or safety constraints — these must survive verbatim into the digest.

**Compression safety check (gate item before reset):** Can the task resume correctly from the digest alone, without access to any discarded material? If no, extract more before discarding.

## 5. Budget extended thinking per task type

Extended thinking spends output tokens on reasoning traces before the final answer.

**Enable it when:**
- The task requires multi-step logical deduction
- The model produces inconsistent or wrong answers without visible reasoning
- The problem space is underdetermined and requires exploring multiple paths

**Cap or disable it when:**
- The task is well-defined with an unambiguous answer
- The workload is latency-sensitive
- The prompt already scaffolds reasoning — chain-of-thought in the prompt plus extended thinking double-spends reasoning tokens

Set a reasoning token budget matched to the task complexity level. Never leave it uncapped. Refer to current API docs for model-specific parameters, as they change with each release.

## 6. Generate the shortest complete output

Output tokens cost roughly 5x input tokens. Every output token not generated is the cheapest output token.

Do not include: restatements of the question, generic introductions, conclusions that merely repeat the answer, unrequested background, filler phrases, apologies, or repetitive examples.

Prefer: **Answer → necessary explanation → necessary caveat**

**Mandatory pruning step:** After drafting any response, go sentence by sentence. For each sentence ask: can this be removed without reducing correctness or usefulness? If yes, remove it. This step is not optional — skipping it means the output discipline rule was not applied. The pruning pass is recorded as item 5 in the section 9 gate.

Format discipline:
- Set `max_tokens` per use case; never default to the model maximum
- Prefer structured outputs when a machine reads the result: `Return JSON: {name, score, reason}` costs fewer tokens than a verbose prose instruction — both in the prompt and in the output it produces
- Match verbosity to the consumer: parsers need accuracy, not explanation; end users need explanation, not JSON
- Set effort/reasoning levels per task type; never globally at maximum

Instruction deduplication: consolidate overlapping instructions. Instead of "be concise, avoid detail, keep it short, don't be verbose" use one: "be concise; include only what completes the task."

## 7. Use tools only when the task requires them

Before invoking any tool, answer both questions:
1. Can this task be solved reliably without this tool?
2. Is this result already available from earlier in this conversation?

If yes to either, do not call the tool. Invoking a tool without first answering both questions is a gate violation.

When a tool is necessary:
- Minimize the input sent to it
- Request only the information the task requires
- Reuse results already obtained in this conversation
- Batch compatible operations when that reduces overhead
- Never call the same tool twice for information already available

## 8. Route to the minimum capable model tier

**Default model: claude-sonnet-4-6** — the most efficient model for the widest range of tasks. Use it unless there is a concrete reason to go lower or higher.

The Claude lineup: Haiku → Sonnet (default: claude-sonnet-4-6) → Opus → Fable. Start at the lowest tier plausible for the classified task level. Escalate only when the current tier demonstrably fails.

**Haiku-class:** Level 0–1 tasks. Classification, extraction, formatting, simple Q&A, short-form summarization.
**Sonnet-class (default):** Level 1–2 tasks. Most coding, analysis, writing, sustained reasoning, agentic loops. Use claude-sonnet-4-6 unless escalation is warranted.
**Opus-class:** Level 2–3 tasks. Complex multi-step reasoning, research synthesis, tasks requiring sustained coherence over long outputs.
**Fable-class:** Level 3 tasks where quality is the dominant variable. Highest latency.

**Escalate only when:** the model fails to satisfy requirements, confidence is insufficient, the task exceeds the tier's reasoning capability, or verification identifies an error. When escalating, transfer only compressed relevant state — not full conversation history.

**De-escalate mid-task:** if a complex task resolves to a simpler subtask on inspection, reclassify and stop processing at the original tier. Do not maintain Level 3 effort on a Level 1 question just because the session started at Level 3.

Verify current model capabilities at https://docs.anthropic.com before routing. `references/pricing-snapshot.md` shows the current ladder for orientation; `references/cost-formulas.md` has the break-even math for escalation and caching decisions.

## 9. Quality-control gate before outputting

Do not send until all four pass:

1. Did I answer the actual request, preserving all constraints (negations, numbers, conditions)?
2. Did I complete the mandatory pruning pass? If no, do it now.
3. For L2+ tasks: did I call advisor before acting AND before declaring done? If no, do so before sending.
4. Is context approaching 180k tokens? If yes, run the section 4 compression protocol first.
