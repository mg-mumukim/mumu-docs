<!-- Sources:
- note/source/2026-04-14-serving-options-docs-template.md (current Google Doc snapshot)
- note/source/2026-04-14-serving-options-planning-meeting-notion.md (planning meeting 2026-04-14)
- note/source/2026-04-14-serving-options-gen-ai-mg-ai-meeting-gdocs.md (MGAI x Tinder sync 2026-04-13)
- LP-531: Prepare serving option document headers (structuring)
- note/context/2026-04-09-mumu-profile.md
-->

## Changes from v2
- Replaced Appendix structure with Product / Engineering top-level split
- Moved user flow, edge cases, and open product decisions under Product Considerations
- Moved benchmarks, cost estimation, and open technical questions under Engineering Evidence
- Pros/Cons/Suggestion remain in the overview table as the synthesis of both sides

# Photo Review Serving Options — Proposed Document Structure

This document proposes a revised structure for the "Photo Review Serving Optimizations & Remaining Questions" Google Doc so that Product and Engineering stakeholders can make a serving option decision.

## 1. Executive Summary

One-paragraph overview of the decision to be made, the three options, and the recommended path. Written last after all other sections are complete.

## 2. Constraints and Scope

Fixed boundaries that all options must satisfy.

- MVP scope: English regions (US, CAN, AUS, NZL)
- Budget ceiling: $50K/month for test regions
- Coaching generation latency requirements per option
- Gemini API availability and QoS dependency

## 3. Options Overview

Side-by-side comparison table. Serves as the index into the rest of the document; each row's detail lives in the corresponding section below.

|  | Option 1: Before App Open | Option 2: On App Open, >= 30s | Option 3: On App Open, <= 5s |
|--|---------------------------|-------------------------------|------------------------------|
| Approach | Gemini Batch API precompute | Gemini API near-real-time precompute (1m model) | True real-time generation |
| Ideal Trigger | Scheduled batch | App open | App open |
| Latency to Result | 0s or unavailable | 30-60s | 2-5s |
| Nudge to Profile Home | (existing) | (existing) | (existing) |
| Notification Timing | (existing) | (existing) | (existing) |
| Dependency | (existing) | (existing) | (existing) |
| UX Consistency | Hit or miss | Consistent but slower | Most consistent |
| User Flow | see 4.1 | see 4.1 | see 4.1 |
| Edge Cases | see 4.2 | see 4.2 | see 4.2 |
| Cost Estimation | see 5.2 | see 5.2 | see 5.2 |
| Pros | (list) | (list) | (list) |
| Cons | (list) | (list) | (list) |
| Confidence in 6 weeks | High / Medium / Low | High / Medium / Low | High / Medium / Low |
| Suggestion | (conditional recommendation) | (conditional recommendation) | (conditional recommendation) |

## 4. Product Considerations

### 4.1 User Flow per Option

Simplified sequence diagram per option (user, client, server, Gemini/GPU, storage) with timing annotations at each step. One diagram per option.

- Option 1: Before App Open
- Option 2: On App Open, >= 30s
- Option 3: On App Open, <= 5s

### 4.2 Edge Cases and Fallback Scenarios

Per option: which users are missed or degraded, estimated volume, fallback plan, UX impact.

Example for Option 1: users who sign up or change photos mid-batch cycle are not covered until the next batch run.

### 4.3 Open Product Decisions

Items that stakeholders need to decide. Each states the decision, the options, and how it affects downstream work.

- Initial deployment target population
- In-app nudge design and trigger logic
- Adoption rate assumption for cost modeling
- Coaching message refresh policy: same message until user acts vs. rotating across sessions

## 5. Engineering Evidence

Empirical data to resolve technical uncertainty that blocks the decision. Directly addresses concerns raised in the MGAI x Tinder sync (2026-04-13).

### 5.1 Benchmarks

#### Gemini Batch API
- Throughput (requests/hour), turnaround time, error handling, throttling behavior
- Validates Option 1 feasibility

#### L40S + S3 Image Download
- Throughput (profiles/sec) and per-request latency
- Batch mode: throughput matters. Real-time mode: latency matters.
- Validates all options

#### Model Pipeline PoC
- Comparison: pure Gemini Pro vs. Qwen + Gemini Flash Light
- Metrics: latency, score, token usage
- Validates Option 3 feasibility and cost impact across options
- Evaluation methodology to be defined before execution

### 5.2 Cost Estimation

Single-number cost per (region x option). Current preliminary estimate:

| Scope | Option 1 (Batch) | Option 2 (Near-RT) |
|-------|-------------------|---------------------|
| Test regions (US+CAN+AUS) | $33K/mo | $70K/mo |
| Global | $163K/mo | $329K/mo |

Full breakdown by cost component (LLM API, GPU node) to be updated with benchmark results.

### 5.3 Open Technical Questions

Retain existing Question / Status / Note table from current Google Doc, merged with items from meeting notes.

Known items:
- Profile photo change cutoff time for batch inclusion (Option 1)
- Gemini batch throughput and scaling reliability (NanoBanana precedent)
- Token savings from adding Qwen to the pipeline
