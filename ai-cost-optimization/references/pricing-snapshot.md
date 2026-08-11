# Pricing snapshot — July 2026 (STALE BY DEFAULT)

Snapshot taken July 30, 2026 from third-party trackers that verified against Anthropic's live pricing page on July 26, 2026. This file exists only for orientation — to show the shape of the ladder and typical multipliers. NEVER quote these figures as current. Re-verify at https://claude.com/pricing before any recommendation.

## Claude model ladder (USD per MTok, standard API)

| Model | Input | Cached read | Output | Notes |
|---|---|---|---|---|
| Claude Haiku 4.5 | $1.00 | $0.10 | $5.00 | |
| Claude Sonnet 5 | $2.00 | $0.20 | $10.00 | Promo through Aug 31, 2026; $3/$15 from Sept 1 |
| Claude Sonnet 4.6 | $3.00 | $0.30 | $15.00 | |
| Claude Opus 5 | $5.00 | $0.50 | $25.00 | Launched Jul 24, 2026; topped Artificial Analysis index at launch |
| Claude Opus 4.8 | $5.00 | $0.50 | $25.00 | |
| Claude Fable 5 | $10.00 | $1.00 | $50.00 | Mythos-class top tier |

## Structural facts (more durable than prices, still verify)

- Cache reads ≈ 0.1x input; cache writes 1.25x (5-min TTL) or 2x (1-hour TTL).
- Batch API: flat 50% off input and output.
- Full 1M-token context at standard rates on Fable 5, Opus 4.6+, Sonnet 4.6+ — no long-context premium.
- Fast mode: 2x base pricing (Claude API only).
- US-only inference residency: 1.1x multiplier on eligible models.
- Models from Opus 4.7 onward use a tokenizer producing up to ~35% more tokens for the same text vs older generations.
