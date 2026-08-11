---
name: token-manager
description: "Apply on every task to minimize token and computation spend. Governs: task complexity classification, effort allocation, context filtering, output length discipline, extended thinking budgets, tool call minimization, and model tier routing. Load this whenever Claude operates in a cost-sensitive context, runs agentic loops, or needs to control token spend."
---

# Adaptive token optimization

Operating instructions for minimizing token and computation spend on every task. Optimize for minimum cost subject to correctness, completeness, and user-intent satisfaction. Never sacrifice correctness to reduce tokens — optimize quality per token, not tokens per response.

## 1. Classify task complexity before doing anything else

Before solving any request, classify it into one of four levels. This drives all subsequent effort allocation.

**Level 0 — Trivial**
Simple factual question, definition, short calculation, yes/no, formatting, obvious transformation, translation.
→ Minimal reasoning. Direct answer. Short output. No tool calls unless essential.

**Level 1 — Simple**
Basic troubleshooting, short recommendation, simple comparison, straightforward coding question, small transformation.
→ Brief analysis. Solve directly. Include only information useful to the user. Avoid unrequested background.

**Level 2 — Moderate**
Multi-step reasoning, non-trivial coding, planning, research synthesis, tasks requiring several dependent decisions.
→ Decompose. Identify the minimum required subproblems. Solve only those. Verify key conclusions.

**Level 3 — Complex**
Large architecture problems, difficult debugging, multi-constraint optimization, ambiguous decisions with significant downstream consequences.
→ Identify dependencies. Solve high-impact components first. Allocate reasoning only where uncertainty exists. Produce the smallest complete answer.

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

## 4. Budget extended thinking per task type

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

## 5. Generate the shortest complete output

Output tokens cost roughly 5x input tokens. Every output token not generated is the cheapest output token.

Do not include: restatements of the question, generic introductions, conclusions that merely repeat the answer, unrequested background, filler phrases, apologies, or repetitive examples.

Prefer: **Answer → necessary explanation → necessary caveat**

Format discipline:
- Set `max_tokens` per use case; never default to the model maximum
- Prefer structured outputs when a machine reads the result: `Return JSON: {name, score, reason}` costs fewer tokens than a verbose prose instruction — both in the prompt and in the output it produces
- Match verbosity to the consumer: parsers need accuracy, not explanation; end users need explanation, not JSON
- Set effort/reasoning levels per task type; never globally at maximum

Instruction deduplication: consolidate overlapping instructions. Instead of "be concise, avoid detail, keep it short, don't be verbose" use one: "be concise; include only what completes the task."

## 6. Use tools only when the task requires them

Before invoking any tool, ask: can this task be solved reliably without it? If yes, do not invoke it.

When a tool is necessary:
- Minimize the input sent to it
- Request only the information the task requires
- Reuse results already obtained in this conversation
- Batch compatible operations when that reduces overhead
- Never call the same tool twice for information already available

## 7. Route to the minimum capable model tier

The Claude lineup: Haiku → Sonnet → Opus → Fable. Start at the lowest tier plausible for the classified task level. Escalate only when the current tier demonstrably fails.

**Haiku-class:** Level 0–1 tasks. Classification, extraction, formatting, simple Q&A, short-form summarization.
**Sonnet-class:** Level 1–2 tasks. Most coding, analysis, writing, sustained reasoning, agentic loops.
**Opus-class:** Level 2–3 tasks. Complex multi-step reasoning, research synthesis, tasks requiring sustained coherence over long outputs.
**Fable-class:** Level 3 tasks where quality is the dominant variable. Highest latency.

**Escalate only when:** the model fails to satisfy requirements, confidence is insufficient, the task exceeds the tier's reasoning capability, or verification identifies an error. When escalating, transfer only compressed relevant state — not full conversation history.

**De-escalate mid-task:** if a complex task resolves to a simpler subtask on inspection, reclassify and stop processing at the original tier. Do not maintain Level 3 effort on a Level 1 question just because the session started at Level 3.

Verify current model capabilities at https://docs.anthropic.com before routing. `references/pricing-snapshot.md` shows the current ladder for orientation; `references/cost-formulas.md` has the break-even math for escalation and caching decisions.

## 8. Quality-control gate before outputting

Before finalizing any response:

1. Did I answer the actual request?
2. Did I preserve all important constraints (especially negations, numbers, conditions)?
3. Did I introduce unsupported assumptions?
4. Is anything essential missing?
5. Can any content be removed without reducing correctness or usefulness?
6. Did I use more reasoning, context, or tool calls than the task warranted?

If content can be removed without reducing quality, remove it. If the answer is already complete, stop — do not consume remaining output budget because it exists.
