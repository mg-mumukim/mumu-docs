| Key           | Value                          |
| ------------- | ------------------------------ |
| Source        | `https://docs.google.com/document/d/1Q4x0DwGJuDGjTaqFAtlbHFH-KD1cWZosPI-zdhFTQ9U/edit?tab=t.zbd36zecf8rg` |
| Downloaded at | `2026-04-22 17:05:00`        |

---

# Serving Options

Maintainer [Juniper Han](mailto:juniper.han@match.com) [Brandon Choi](mailto:brandon.choi@match.com)  
Reviewer [Mumu Kim](mailto:mumu.kim@match.com)  
Last updated 2026-04-15  
---

[Part 1: Decision Summary (for all stakeholders)](#part-1:-decision-summary-\(for-all-stakeholders\))

[At a glance](#at-a-glance)

[Recommendation](#recommendation)

[Open questions for PM](#open-questions-for-pm)

[Part 2: Product Details (for PM and product stakeholders)](#part-2:-product-details-\(for-pm-and-product-stakeholders\))

[How users experience each option](#how-users-experience-each-option)

[User coverage comparison](#user-coverage-comparison)

[Remaining product questions](#remaining-product-questions)

[Part 3: Engineering Details (for engineers)](#part-3:-engineering-details-\(for-engineers\)-[wip])

[Architecture](#architecture)

[Option 1: Batch flow](#option-1:-batch-flow)

[Option 2: On-demand flow](#option-2:-on-demand-flow)

[Timeline](#heading=h.3feo6fqm9zg3)

[Measured numbers](#measured-numbers)

[Gemini Flash-Lite sync latency](#gemini-flash-lite-sync-latency)

[Gemini Batch turnaround](#gemini-batch-turnaround)

[Cost](#cost)

[Data not yet collected](#data-not-yet-collected)

[Risk comparison](#risk-comparison)

[Appendix](#appendix)

[A. Why True Realtime (under 5s) was eliminated](#a.-why-true-realtime-\(under-5s\)-was-eliminated)

---

# Part 1: Decision Summary (for all stakeholders) {#part-1:-decision-summary-(for-all-stakeholders)}

## At a glance {#at-a-glance}

All inference runs through Gemini API (no self-hosted GPU/ML). True realtime (under 5s) was eliminated because Gemini p50 is already 17.5s for our coaching payload. Two options remain.

|  | Option 1: Batch Precompute | Option 2: On-demand Sync |
| :---- | :---- | :---- |
| **How it works** | Run hourly batch via Gemini Batch API (turnaround up to 24h per Google SLA, observed \~17min in dry run). Store result. Serve instantly when user opens Profile Home. | Start generating coaching when user opens the app. By the time they reach Profile Home, result may be ready. |
| **User experience** | Nudge to Profile Home appears immediately at app open. User acts on it while intent is fresh. | Generation starts at app open but result takes \~30s. By then user is likely mid-swipe. Nudging at that point interrupts the core experience and users are unlikely to act on it. |
| **Cost (test region)** | \~$33K/mo (within $50K budget) | \~$66K/mo upper bound (exceeds $50K budget) |
| **Cost (global)** | \~$163K/mo | \~$325K/mo upper bound |
| **Budget** | $50K/mo cap. Option 1 fits. | $50K/mo cap. Option 2 exceeds unless actual trigger rate is lower than assumed, or cache discount reduces cost enough. Needs Kevin's verification. |
| **Time to ship** | MGAI \+ SOUP parallel (contract finalized). 5-6 wks \+ 1 wk QA. | Same. 5-6 wks \+ 1 wk QA. |
| **Biggest risk** | New/returning users get nothing until next batch. Coverage is non-deterministic. | Nudge arrives \~30s after app open when user is already swiping. Interrupting the swipe flow to send them to Profile Home is a poor experience with low conversion. |
| **Privacy** | Stores coaching per user. Triggers retention/deletion obligations. | Ephemeral. No stored inferences. |
| **Infra complexity** | Batch pipeline \+ coaching store \+ scheduler \+ cache invalidation | Single request-response endpoint |

## Recommendation {#recommendation}

**Option 2 (On-demand Sync) as the MVP serving path**, with caveats below.

Reasons, in order of criticality:

1. **Covers every user.** No cold-start gap, no batch-miss. Every app open triggers coaching generation. Option 1's precompute set is bounded, so some users always fall through.  
2. **No privacy/retention complexity.** No stored inferences means no deletion pipeline, no audit log, no Legal review on data retention.  
3. **Simpler to build and operate.** One endpoint. No batch scheduler, no coaching store, no cache invalidation logic.  
4. **Latency is within budget.** p95 \= 25.0s fits the 30s budget. Timeout handling is needed for \~1% at p99 (30.5s).

**However, Option 1 has a strong UX advantage:** coaching is ready at app open, so the nudge to Profile Home happens immediately while the user's attention is uncommitted. In Option 2, the \~30s delay means the nudge arrives when the user is mid-swipe, which is disruptive and unlikely to convert. If nudge timing is the highest priority, Option 1 is the better path despite its infrastructure overhead.

**Caveats that need product resolution:**

- **UX: nudge timing is the core weakness.** In Option 1, coaching is already ready at app open, so the nudge to Profile Home happens immediately while the user's attention is uncommitted. In Option 2, coaching takes \~30s to generate. By that time the user is mid-swipe, and interrupting with a "check your profile coaching" nudge is disruptive and unlikely to convert. This timing gap is Option 2's strongest disadvantage compared to Option 1\.  
- **Cost: roughly 2x Option 1\.** At global scale, this is \~$325K/mo vs \~$163K/mo. However, Option 1 also pays for users who never view the result, so the effective cost gap may be smaller. The actual Option 2 cost depends on Profile Home view frequency, which is not yet measured.  
- **Hybrid is possible later.** If Option 2 launches first, adding a precompute cache (for cost reduction or instant UX) is a backward-compatible upgrade. The reverse is harder.

## Open questions for PM {#open-questions-for-pm}

| Question | Impact on decision |
| :---- | :---- |
| Is a delayed nudge (\~30s after app open, mid-swipe) acceptable, or must coaching be surfaced at app entry? | If must be at app entry, Option 1 is the only viable path |
| Monthly budget is $50K/mo. Can Option 2 fit within it? | At current estimate ($66K), no. Needs lower trigger rate or cache discount to close the gap.  |

---

# Part 2: Product Details (for PM and product stakeholders) {#part-2:-product-details-(for-pm-and-product-stakeholders)}

## How users experience each option {#how-users-experience-each-option}

**Option 1: Batch Precompute.** A scheduled job generates coaching for eligible users ahead of time. When a user opens Profile Home, the coaching is already there. If the user was not in the batch (new user, changed profile after last batch), they see no coaching until the next cycle.

**Option 2: On-demand Sync.** Coaching generation starts when the user opens the app. After \~30 seconds, the result is ready. If the user navigates to Profile Home after that, coaching appears instantly. If they visit Profile Home before generation completes, they may see it appear during their visit. If they never visit Profile Home during the session, the generated coaching goes unused.

## User coverage comparison {#user-coverage-comparison}

| Scenario | Option 1 | Option 2 |
| :---- | :---- | :---- |
| Returning user, no profile change | Coaching ready instantly | Generation starts at app open. Ready by the time user reaches Profile Home. |
| User changed profile since last batch | No coaching until next batch | Fresh coaching generated at app open |
| Brand new user (first day) | No coaching until first batch | Coaching generated at app open |
| User opens Profile Home within 30s of app open | Instant if cached | Coaching not yet ready. Either wait or show on return. |
| User opens Profile Home after 30s | Instant if cached | Coaching already generated. Appears immediately. |
| User never visits Profile Home in session | Coaching stored but unused | Coaching generated but unused (wasted call) |
| Power user (5x/day) | Same cached coaching each time | Fresh call each app open (higher cost) |

## Remaining product questions {#remaining-product-questions}

| Question | Owner | Note |
| :---- | :---- | :---- |
| What adoption rate for cost estimation? 20%? | Juniper / Jaini |  |
| Leave same coaching until dismissed, or rotate messages? | Juniper / Jaini |  |
| Cut-off time for profile updates to be included in next batch? | Juniper / Jaini | Option 1 only |
| Bottom sheet appears once or resurfaces? | Juniper |  |
| How many hours/days can coaching stay fresh? | Juniper \+ Jaini | Determines Option 1 batch cadence |
| Non-Profile-Home entry points planned? | Juniper | Option 1 benefits if yes |

---

# Part 3: Engineering Details (for engineers) \[WIP\] {#part-3:-engineering-details-(for-engineers)-[wip]}

## Architecture {#architecture}

SOUP calls POST /v1/photo-review to AURA. AURA decides internally whether to serve from cache (Option 1\) or call Gemini live (Option 2). SOUP does not interact with the Coaching Store directly.

| Component | Owner | Role |
| :---- | :---- | :---- |
| Tinder Client | Tinder | Renders Profile Home and coaching bottom sheet |
| SOUP Backend | Tinder (Henry Au, Xincen Hao) | Owns profile data, calls AURA API, handles user-facing response |
| AURA ML API | MGAI (Brandon) | Builds Gemini payload, returns coaching JSON |
| Gemini API | Google (vendor) | Vision \+ language call. Sync and Batch paths. |
| Coaching Store | MGAI, Option 1 only | Precomputed coaching keyed by (user\_id, profile\_version) |

### Option 1: Batch flow {#option-1:-batch-flow}

**Precompute phase (runs periodically):** Scheduler triggers batch. SOUP provides eligible user list. AURA submits to Gemini Batch API. Results return in \~17 minutes. AURA writes coaching to Coaching Store.

**Read phase (user opens Profile Home):** SOUP calls AURA (same endpoint as Option 2). AURA checks Coaching Store. Hit: return cached coaching (\~100ms). Miss: return empty, or fall back to sync generation (hybrid mode).

### Option 2: On-demand flow {#option-2:-on-demand-flow}

**Triggered at app open:** User opens the Tinder app. SOUP fires a coaching request to AURA in the background. AURA calls Gemini sync API. Gemini returns coaching in \~30s. AURA returns to SOUP. Result is held in memory.

**When user reaches Profile Home:** If coaching is already generated: show immediately. If still in progress: show when ready (or on next visit). If 30s timeout exceeded: no coaching for this session.

**Key design question:** What if the user never visits Profile Home during this session? Options: (a) nudge toward Profile Home via bottom sheet or badge, (b) hold result for the next session, (c) accept the wasted call as cost of coverage.

## Measured numbers {#measured-numbers}

All numbers below are measured, not estimated.

### Gemini Flash-Lite sync latency {#gemini-flash-lite-sync-latency}

Source: Prompt Factory database (generation\_snapshots), model gemini/gemini-3.1-flash-lite-preview, prompt v4+, \# of samples=2,486 coaching calls, 2026-04-20 to 2026-04-22.

| Metric | Value |
| :---- | :---- |
| p50 | 17.5s |
| p90 | 22.4s |
| p95 | 25.0s |
| p99 | 30.5s |
| avg input tokens | 18,451 |
| avg output tokens | 3,384 |

### Gemini Batch turnaround {#gemini-batch-turnaround}

Source: LP-532 dry run (2026-04-14), gemini-2.5-pro via Vertex Batch, 9 batch request, 25 coaching calls.

| Metric | Value |
| :---- | :---- |
| Observed median (dry run) | 16.8 min |
| Observed range (dry run) | 9.9 to 20.9 min |
| Google SLA | Best-effort, up to 24 hours. No committed turnaround. |
| Cost saving vs sync | 50% |

The 17-minute observed turnaround is from a small-scale dry run (25 requests) and is not guaranteed at production volume. Design assumes hourly batch cadence with up to 24h worst-case per Google SLA. Hourly batches reduce the staleness window: a user who changes their profile gets fresh coaching within \~1 hour in the typical case.

### Cost {#cost}

| Item | Value | Source |
| :---- | :---- | :---- |
| Per-call cost (sync) | $0.0086 | LLM Cost section (Mumu) |
| Per-call cost (batch, 50% off) | $0.0043 | Gemini Batch API pricing |
| Test region DAU | 3.65M (USA+CAN+AUS) | Kevin, Databricks |
| Global DAU | 18M | Kevin, Databricks |
| Steady-state trigger rate | \~7% of DAU/day | Kevin's analysis |
| Monthly calls (test region) | 7.67M | 3.65M x 7% x 30 |
| Monthly calls (global) | 37.8M | 18M x 7% x 30 |
| Option 1 monthly, test region | \~$33K | 7.67M x $0.0043 |
| Option 1 monthly, global | \~$163K | 37.8M x $0.0043 |
| Option 2 monthly, test region (upper bound) | \~$66K | 7.67M x $0.0086 |
| Option 2 monthly, global (upper bound) | \~$325K | 37.8M x $0.0086 |
| Day-1 cold start volume (test region) | 3.65M | One-time |
| Day-1 cold start volume (global) | 18M | One-time |
| Ramp-down to steady state | \~21 days | Kevin's Mode report |

## Data not yet collected {#data-not-yet-collected}

| Data | Owner | Why it matters |
| :---- | :---- | :---- |
| Profile Home view frequency (views per DAU per day) | Juniper, Harriet (ask Tinder data team) | Determines actual Option 2 cost. Current $66K/$325K is upper bound assuming every trigger \= one view. |
| Profile edit to Profile Home re-view rate | Jaini (ask Tinder data team) | Measures how often users check Profile Home right after editing. High rate means Option 1 staleness hurts more. |
| Gemini sync QPS quota (Tinder account) | Brandon, Mumu | Confirms Option 2 can handle peak traffic without throttling. |
| SOUP backend LOE per option | Brandon (ask Henry Au) | Validates whether SOUP effort differs between options. |
| Privacy/Legal review on stored inferences | Mumu (ask Blake) | Determines Option 1 privacy overhead. |
| Monthly budget cap | Juniper, Kevin Celustka | Determines if Option 2 cost is acceptable. |

## Risk comparison {#risk-comparison}

| Risk | Option 1 | Option 2 |
| :---- | :---- | :---- |
| **Kills the option if...** | Batch turnaround exceeds staleness tolerance | Gemini p95 exceeds 30s (currently 25s, passes) |
| **Biggest operational risk** | Batch failure, partial job completion | QPS throttling at peak hours |
| **UX risk** | Some users get no coaching (batch miss) | User may never visit Profile Home; coaching unused |
| **Privacy risk** | Stored inferences need retention/deletion | None (ephemeral) |
| **Cost risk** | Lower (batch discount) | Higher (scales with views) |

---

# Appendix {#appendix}

## A. Why True Realtime (under 5s) was eliminated {#a.-why-true-realtime-(under-5s)-was-eliminated}

All inference runs through Gemini API without self-hosted ML. The coaching payload is 9 photos \+ system prompt \+ KB \+ bio, totaling \~18,000 input tokens.

1. **Measured p50 is 17.5s.** No path to 5s without a self-hosted concept extractor.  
2. **Google does not offer latency SLA.** Gemini guarantees availability only, not response time.  
3. **No retry budget at 5s.** A single timeout leaves zero fallback time.  
4. **The original 5s option assumed self-hosted GPU** (Qwen3-VL under 1s for concept extraction, LLM for text only). Without GPU serving, this architecture is not possible.

Revival requires either reintroducing self-hosted GPU or Google guaranteeing under 3s p95 for 9-photo vision calls.  
