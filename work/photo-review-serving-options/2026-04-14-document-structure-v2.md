<!-- Sources:
- note/source/2026-04-14-serving-options-docs-template.md (current Google Doc snapshot)
- note/source/2026-04-14-serving-options-planning-meeting-notion.md (planning meeting 2026-04-14)
- note/source/2026-04-14-serving-options-gen-ai-mg-ai-meeting-gdocs.md (MGAI x Tinder sync 2026-04-13)
- LP-531: Prepare serving option document headers (structuring)
- note/context/2026-04-09-mumu-profile.md
-->

## Changes from v1
- Consolidated 9 sections into 5 sections
- Merged user flow, edge cases, and pros/cons into the main options comparison table as new rows
- Removed owner annotations from all sections
- Moved engineering benchmarks and cost estimation into a single evidence appendix

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

## 3. Options Comparison

The core of the document. A single expanded table comparing all three options across every dimension needed for a decision.

|  | Option 1: Before App Open | Option 2: On App Open, >= 30s | Option 3: On App Open, <= 5s |
|--|---------------------------|-------------------------------|------------------------------|
| Approach | Gemini Batch API precompute | Gemini API near-real-time precompute (1m model) | True real-time generation |
| Ideal Trigger | Scheduled batch | App open | App open |
| Latency to Result | 0s or unavailable | 30-60s | 2-5s |
| Nudge to Profile Home | (existing content) | (existing content) | (existing content) |
| Notification Timing | (existing content) | (existing content) | (existing content) |
| Dependency | (existing content) | (existing content) | (existing content) |
| UX Consistency | Hit or miss | Consistent but slower | Most consistent |
| **User Flow** | (sequence diagram + steps) | (sequence diagram + steps) | (sequence diagram + steps) |
| **Edge Cases** | (missed users, volume estimate, fallback plan) | (missed users, volume estimate, fallback plan) | (missed users, volume estimate, fallback plan) |
| **Cost Estimation** | (from Appendix) | (from Appendix) | (from Appendix) |
| **Pros** | (list) | (list) | (list) |
| **Cons** | (list) | (list) | (list) |
| **Confidence in 6 weeks** | High / Medium / Low | High / Medium / Low | High / Medium / Low |
| **Suggestion** | (conditional recommendation) | (conditional recommendation) | (conditional recommendation) |

Bold rows are new rows to be added to the existing table. The rest already exist in the current Google Doc.

Notes on new rows:
- **User Flow**: Simplified sequence diagram per option (user, client, server, Gemini/GPU, storage) with timing annotations. Primary input for Product decision-making.
- **Edge Cases**: Which users are missed or degraded, estimated volume, fallback scenario, UX impact. Example for Option 1: users who sign up or change photos mid-batch cycle are not covered until the next batch run.
- **Cost Estimation**: Single-number summary from Appendix. Current preliminary estimate:

| Scope | Option 1 (Batch) | Option 2 (Near-RT) |
|-------|-------------------|---------------------|
| Test regions (US+CAN+AUS) | $33K/mo | $70K/mo |
| Global | $163K/mo | $329K/mo |

## 4. Open Items

Decisions beyond the serving option choice that stakeholders need to make, plus tracked questions. Each item states what the decision is and how it affects downstream work.

Serving-related:
- Adoption rate assumption for cost modeling (currently open)
- Coaching message refresh policy: same message until user acts vs. rotating across sessions
- Profile photo change cutoff time for batch inclusion (Option 1)

Product-related:
- Initial deployment target population
- In-app nudge design and trigger logic

Technical questions (retain existing Question / Status / Note table from current doc, merged with items from meeting notes).

## 5. Appendix: Engineering Evidence

Supporting data for the cost and feasibility claims in Section 3. Addresses Trideep's concerns from the 2026-04-13 sync.

### 5.1 Gemini Batch API Benchmark
- Throughput (requests/hour), turnaround time, error handling, throttling behavior
- Validates Option 1 feasibility

### 5.2 L40S + S3 Image Download Benchmark
- Throughput (profiles/sec) and per-request latency
- Batch mode: throughput. Real-time mode: latency.
- Validates all options

### 5.3 Model Pipeline PoC
- Comparison: pure Gemini Pro vs. Qwen + Gemini Flash Light
- Metrics: latency, score, token usage
- Validates Option 3 feasibility and cost impact across options
- Evaluation methodology to be defined before execution

### 5.4 Cost Estimation Detail
- Full breakdown by region, option, and cost component (LLM API, GPU node)
- Updates the preliminary numbers once benchmarks complete
