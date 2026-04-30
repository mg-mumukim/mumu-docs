# SOUP Photo Feedback — API Volume & LLM Cost Estimation

**Author:** Ben Shin (ML Team)
**Date:** 2026-03-25
**Purpose:** Estimate the expected Frontier LLM API call volume and cost for the SOUP Photo Feedback project.

## Trigger Logic
1. **Every app open** → check if user needs LLM evaluation
2. **First-time user (no prior feedback)** → trigger LLM eval
3. **Returning user + photos changed since last eval** → re-trigger LLM eval
4. **Returning user + photos NOT changed** → skip (use cached result)

## Key Questions
- How many LLM API calls per day at steady state?
- What is the ramp-up cost (cold start for all existing DAU)?
- What does this cost at Frontier LLM pricing?

## 7. LLM API Cost Estimation

### Assumptions
- Each photo ≈ 1,000 image input tokens + 200 text context tokens
- Output per evaluation ≈ 1,000 tokens (feedback + scores)
- Weighted avg photos per user ≈ 5 (from distribution above)
- Per-user eval ≈ **5,000 input tokens + 1,000 output tokens**

### Pricing Models
| Model | Input ($/1M tokens) | Output ($/1M tokens) |
|-------|--------------------:|---------------------:|
| GPT-4o | $2.50 | $10.00 |
| GPT-4.1-mini | $0.40 | $1.60 |
| Claude Sonnet 4 | $3.00 | $15.00 |
| Gemini 2.5 Flash | $0.15 | $0.60 |

## 8. Summary & Recommendations

### Key Findings

| Metric | Value |
|--------|-------|
| DAU (global) | ~18.5M |
| Total daily app opens | ~460-510M |
| Avg app opens per user/day | ~25 (median: 8) |
| Users changing photos/day | ~1.6M (~8.5% of DAU) |
| Avg photos per active user | ~5 |

### Cost Summary (Steady State — Photo Changers Only)

| Model | Daily Cost | Monthly Cost |
|-------|-----------|-------------|
| GPT-4o | ~$26K | ~$780K |
| GPT-4.1-mini | ~$4.2K | ~$125K |
| Claude Sonnet 4 | ~$32K | ~$960K |
| Gemini 2.5 Flash | ~$2.2K | ~$65K |

### Recommendations

1. **Cache aggressively** — 365-day TTL like Pluto. Steady-state cost is 10x lower than ramp-up.
2. **Phased rollout** — Start with Men-only in English-speaking geos (Android first) to validate.
3. **Model selection matters** — Gemini 2.5 Flash is ~15x cheaper than GPT-4o at steady state.
4. **Batch per user, not per app open** — Only eval once per user per day max, not every app open.
5. **Re-evaluate only on photo change** — ~8.5% of DAU changes photos daily, keeping steady-state manageable.
6. **Consider Growth W0 learnings** — Pluto service already has infra for photo eval with OpenAI API.
