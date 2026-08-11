# Cost and efficiency formulas

All rates in USD per million tokens (MTok) unless stated. Always plug in freshly verified prices (see SKILL.md Step 0).

## Blended rate

Represents a workload's mix of input and output in one number — useful for comparing model tiers on real workloads.

```
blended = (input_ratio x input_rate + output_ratio x output_rate) / (input_ratio + output_ratio)
```

Common mixes:
- Chat / RAG: 3:1 input:output → blended = (3 x In + 1 x Out) / 4
- Agentic loops: often 1:1 or output-heavy → blended = (In + Out) / 2, or measure your own ratio
- Bulk summarization: input-heavy, 10:1 or more

Always state the mix used; a 3:1 blended figure understates agent costs badly.

## Cache-adjusted effective input rate

```
effective_input = hit_rate x cache_read_rate + (1 - hit_rate) x input_rate + write_frequency x write_premium
```

Where `cache_read_rate` ≈ 0.1 x input_rate on Claude. With an 80% hit rate, effective input ≈ 0.28x the headline rate (ignoring writes). This is why a cached higher-tier model can undercut an uncached lower-tier model.

## Cache TTL break-even

A longer TTL costs a higher write premium. It pays when:

```
expected_reuses_within_TTL x (input_rate - cache_read_rate) > (long_TTL_write_rate - short_TTL_write_rate)
```

If the prefix is reused fewer than a handful of times per window, use the short TTL or no cache.

## Efficiency per completed task

```
cost_per_task = avg_attempts x (input_tokens x effective_input + output_tokens x output_rate) / 1,000,000
```

Compare models on this, never on per-token rates in isolation. Measure `avg_attempts` (including retries, self-corrections, and human re-prompts) from real traces or the golden eval set.

## Model escalation break-even

A higher-tier model is justified when:

```
attempts_lower x tokens_lower x rate_lower > attempts_higher x tokens_higher x rate_higher
```

Higher-tier models often use fewer total tokens per task (less flailing), so compare measured token totals — not identical token assumptions — across tiers.

## Tokenizer migration adjustment

When moving to a model generation with a different tokenizer:

```
new_token_count = count via token-counting endpoint on real prompts (do not multiply old counts by a guessed factor)
```

Budget comparisons made with old counts against new rates are the most common source of migration surprises. The tokenizer shift from pre-Opus-4.7 to post-Opus-4.7 generations is up to ~35% more tokens for the same text.

## Worked example template

State assumptions explicitly:

```
Workload: support-ticket triage
Volume: 500K requests/month
Tokens: 4,000 in / 300 out per request
Mix: 13:1 → blended dominated by input
Cache: 3,500-token stable prefix, est. 85% hit rate
Candidate: Haiku-class at $X in / $Y out (verified <date>)
Effective input = 0.85 x 0.1X + 0.15 x X = 0.235X
Monthly ≈ 500K x (4,000 x 0.235X + 300 x Y) / 1M
Avg attempts: 1.1 (measured from eval set)
```

Label every number as measured or estimated.
