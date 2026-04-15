<!-- Sources:
- note/source/2026-04-14-serving-options-docs-template.md (current Google Doc snapshot)
- note/source/2026-04-14-serving-options-planning-meeting-notion.md (planning meeting 2026-04-14)
- note/source/2026-04-14-serving-options-gen-ai-mg-ai-meeting-gdocs.md (MGAI x Tinder sync 2026-04-13)
- LP-531: Prepare serving option document headers (structuring)
- note/context/2026-04-09-mumu-profile.md
-->

# Photo Review Serving Options — Proposed Document Structure

This document proposes a revised structure for the "Photo Review Serving Optimizations & Remaining Questions" Google Doc. The goal is to reorganize the document so that Product and Engineering stakeholders can read it and make a serving option decision with sufficient context and evidence.

## Background

The current Google Doc has five sections: Cost Dashboard, Summary, Constraints, Options (comparison table), and Remaining Questions. Based on the MGAI x Tinder sync (2026-04-13) and the planning meeting (2026-04-14), the following gaps were identified:

- The options table has empty cells for Cost, Pros Summary, Cons Summary, and Suggestion.
- User flows per option are described in text only; no diagrams exist.
- Edge cases (e.g., users who join mid-batch cycle) and fallback scenarios are not documented.
- Engineering benchmark data (Gemini batch throughput, L40S + S3 latency) is pending.
- Decision points beyond serving option choice (deployment target, nudge design) are scattered across meeting notes.

## Proposed Structure

The three options are referred to by their latency profile throughout:

| Label | Description |
|-------|-------------|
| Option 1: Before App Open | Gemini Batch API precompute; result ready or unavailable at app open |
| Option 2: On App Open, >= 30s | Gemini API near-real-time precompute; result ready within 30-60s |
| Option 3: On App Open, <= 5s | True real-time generation; result within 2-5s |

---

### 1. Executive Summary

Purpose: One-paragraph overview of the decision to be made, the three options, and the recommended next steps.

Owner: Mumu Kim (after all sections are filled).

### 2. Constraints and Scope

Purpose: State the fixed boundaries that all options must satisfy.

Contents:
- MVP scope: English regions (US, CAN, AUS, NZL)
- Budget ceiling: $50K/month for test regions
- Coaching generation latency requirements per option
- Dependency on Gemini API availability and QoS

Owner: Mumu Kim.

### 3. Options Overview

Purpose: Side-by-side comparison table for quick scanning.

Retain the current table structure with the following columns filled:

| Row | Status | Owner |
|-----|--------|-------|
| Approach Explained | Done | Brandon |
| Ideal Trigger | Done | Brandon |
| Latency to Result | Done | Brandon |
| Nudge to Profile Home | Done | Brandon |
| Notification Timing | Done | Brandon |
| Dependency | Done | Brandon |
| UX Consistency | Done | Brandon |
| Cost Estimation | Empty — blocked on benchmarks | Mumu |
| Pros Summary | Empty | Juniper |
| Cons Summary | Empty | Juniper |
| Suggestion | Empty | Juniper |

### 4. User Flow per Option

Purpose: For each option, provide a simplified sequence diagram showing what happens from user action to coaching result delivery. This is the primary input Product needs to make a decision.

Sub-sections:
- 4.1 Option 1: Before App Open
- 4.2 Option 2: On App Open, >= 30s
- 4.3 Option 3: On App Open, <= 5s

Each sub-section contains:
- Sequence diagram (user, client, server, Gemini/GPU, storage)
- Step-by-step text description
- Timing annotations at each step

Owner: Brandon (diagrams + processing logic), Juniper (user scenario narrative).

### 5. Edge Cases and Fallback Scenarios

Purpose: Document which users are missed or degraded under each option, quantify the impact, and describe fallback plans.

Contents per option:
- Which users are affected (e.g., new signups during batch processing, users with session < 30s)
- Estimated volume of affected users (from traffic data)
- Fallback scenario (e.g., queue for next batch, push notification on completion)
- UX impact description

Owner: Juniper (case identification + UX impact), Brandon (volume estimation + technical fallback).

### 6. Pros / Cons Comparison

Purpose: Structured comparison to enable a decision. This section is the key deliverable for Tinder Product stakeholders.

Format:

|  | Option 1 | Option 2 | Option 3 |
|--|----------|----------|----------|
| Pros | (list) | (list) | (list) |
| Cons | (list) | (list) | (list) |
| Confidence in 6 weeks | High / Medium / Low | High / Medium / Low | High / Medium / Low |
| Suggestion | (conditional recommendation with decision criteria) | (same) | (same) |

Owner: Juniper (draft), Mumu (review).

### 7. Engineering Evidence

Purpose: Provide empirical data so that engineering uncertainty does not block the decision. This section directly addresses Trideep's concerns from the 2026-04-13 sync.

Sub-sections:

#### 7.1 Gemini Batch API Benchmark
- Throughput (requests/hour), turnaround time, error handling, throttling behavior
- Validates Option 1 feasibility
- Owner: Brandon (execution), Mumu (advisor)

#### 7.2 L40S + S3 Image Download Benchmark
- Throughput (profiles/sec) and per-request latency
- Batch mode: throughput matters. Real-time mode: latency matters.
- Validates all options
- Owner: Brandon (execution), Mumu (advisor)

#### 7.3 Model Pipeline PoC
- Comparison: pure Gemini Pro vs. Qwen + Gemini Flash Light
- Metrics: latency, score, token usage
- Validates Option 3 feasibility and cost impact across options
- Note: evaluation methodology to be defined before execution
- Owner: Brandon (execution), Mumu (advisor)

#### 7.4 Cost Estimation Summary
- Single-number cost per (region x option) table
- Current estimate (from 2026-04-13 sync):

| Scope | Option 1 (Batch) | Option 2 (Near-RT) |
|-------|-------------------|---------------------|
| Test regions (US+CAN+AUS) | $33K/mo | $70K/mo |
| Global | $163K/mo | $329K/mo |

- To be updated with benchmark results
- Owner: Mumu

### 8. Open Decision Points

Purpose: List decisions beyond serving option selection that stakeholders need to make. Presented as a checklist where each item specifies what the decision is, who decides, and how it affects the team's work.

Known items:
- Initial deployment target population
- In-app nudge design and trigger logic
- Adoption rate assumption for cost modeling
- Coaching message refresh policy (same message until action vs. rotating)
- Profile photo change cutoff time for batch inclusion

Owner: Juniper (identification + framing), Brandon (technical dependency annotation).

### 9. Remaining Questions

Purpose: Track open technical and product questions with status.

Retain the current table format (Question / Status / Note). Merge items from the current Google Doc and the planning meeting notes.

Owner: All.

## Section Dependency Map

The following sections block or feed into others:

```
7.1 Gemini Benchmark ──┐
7.2 L40S Benchmark ────┤── 7.4 Cost Estimation ──── 3. Options (Cost row)
7.3 Model PoC ─────────┘                      └──── 6. Pros/Cons
4. User Flow ──────────────── 5. Edge Cases ─────────── 6. Pros/Cons
8. Decision Points ─────────────────────────────────── 1. Executive Summary
```

Sections 2 (Constraints), 4 (User Flow), and 8 (Decision Points) can be drafted immediately. Sections 7.1-7.3 (Benchmarks) require execution time. Section 1 (Executive Summary) is written last.
