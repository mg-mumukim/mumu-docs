# Photo Review — LLM Cost Estimation

> **Sources**:
> - `00_note/2026-04-10-photo-review-llm-cost-estimation.md` (draft outline with reviewer notes)
> - `00_note/2026-04-10-photo-review-kevin-trideep-discussion-traffic.md` (Kevin's traffic model & Slack discussion)
> - `00_note/2026-04-10-photo-review-ben-dbx-notebook.md` (Ben's Databricks notebook: API volume & cost)
> - `01_work/2026-04-09-llm-cost-crosscheck-v1.md` (cross-check analysis, LP-485)
> - AURA Cost Dashboard (referenced, not directly accessed)

---

## TL;DR

At global steady state, Photo Review costs roughly **$163K/month with batch API** or **$325K/month with real-time API**, assuming Gemini 3.1 Flash-Lite at ~$8.6K per million requests. Day-1 cold start adds a one-time spike of ~$77K (batch) or ~$155K (real-time).
Using the batch serving option keeps the monthly cost within the $50K budget only if we scope to test regions first (USA + CAN + AUS steady state: ~$33K/month batch).

[!NOTE] no bold in table.
| Scope | Daily Calls (Steady) | Monthly Cost (Batch) | Monthly Cost (Real-time) |
|-------|--------------------:|---------------------:|-------------------------:|
| Test regions (USA+CAN+AUS) | 255,500 | **$33K** | **$66K** |
| Global | 1,260,000 | **$163K** | **$325K** |

---

## 1. LLM Cost per Request

We use **Gemini 3.1 Flash-Lite Preview** (`gemini/gemini-3.1-flash-lite-preview`) as the baseline model for all cost estimation below.

| #Requests | Cost |
|------|-------|
| 1 | $0.0086 |
| 1M | $8,600 |

This per-request cost is an all-in number that accounts for the typical token usage of a Photo Review call (system prompt, user profile text, multi-image vision tokens, and output coaching text). See the [AURA Cost Dashboard](https://ai-demo.dev.matchgroupcentral.net/aurademo/demo/cost-dashboard) for the detailed breakdown of input/output token parameters or other models.

In practice, cost per request may vary with prompt length, number of photos, photo resolution, and output length. We treat these variations as negligible for planning purposes.

> **Reference**: [Gemini 3.1 Flash-Lite: Built for intelligence at scale](https://blog.google/innovation-and-ai/models-and-research/gemini-models/gemini-3-1-flash-lite/)

---

## 2. Traffic Quantities

### 2.1 DAU Baseline

| Country | DAU |
|---------|----:|
| Global | 18,000,000 |
| USA | 2,800,000 |
| CAN | 450,000 |
| AUS | 400,000 |
| **Test regions total** (USA+CAN+AUS) | **3,650,000** |

### 2.2 When Does a User Trigger an LLM Call?

A user triggers exactly one Photo Review LLM call per day, and only if one of the following conditions is met:

1. **First login after launch (cold start)** — The user has never received a Photo Review score. The very first time they open the app after Photo Review is turned on, the system calls the LLM to generate feedback.
2. **Profile change** — After their initial score, a returning user triggers a new LLM call only when they change their profile (add/delete a photo, change bio, change prompt, add/delete an interest or descriptor).

If a user opens the app but has already been scored and has not changed their profile, **no LLM call is made**. There is a maximum of 1 call per user per day.

### 2.3 Step-by-Step: How Many Calls per Day?

**Step 1 — Day 1 (Cold Start)**

On the very first day Photo Review is turned on, nobody has been scored yet. So every user who opens the app that day will trigger one LLM call.

[!NOTE] no latex formula.
$$\text{Day-1 calls} = \text{DAU} = 18{,}000{,}000 \quad (\text{global})$$

[!NOTE] no metaphor.
Think of it like a new test at school: on the first day, every student has to take it.

**Step 2 — Days 2 through ~21 (Ramp-Down)**

Starting on day 2, users who were already scored on day 1 do NOT trigger a new call (unless they changed their profile). Only two groups of users generate calls:

- **Newly returning users** who missed day 1 and open the app for the first time since launch.
- **Profile changers** who update their photos, bio, prompts, or interests.

Each day, the pool of "never-scored" users shrinks, so the number of first-time calls drops. Kevin's analysis shows this ramp-down takes about **3 weeks** to reach steady state.

**Step 3 — Steady State (After ~3 Weeks)**

Once nearly all active users have been scored at least once, daily calls come from two sources only:

| Source | Rate (% of DAU) | Rationale |
|--------|----------------:|-----------|
| New/returning logins (first-time scores) | ~3% | New registrations + reactivated users who haven't been scored |
| Profile changes | ~4% | Based on Kevin's sample of 1,000 users: ~4% make any profile change on a given day |
| **Total steady-state rate** | **~7%** | Sum of above (with minor overlap, making this a slight overestimate) |

[!NOTE] conservative number should be higher right? the rational seems to be broken.
> **Why 4%, not 8.5%?** Ben's notebook estimates 8.5% of DAU change photos daily. Kevin's analysis, based on a 1,000-user sample, finds ~4% for *any* profile change (photos, bio, prompts, interests, descriptors combined). Since photos are a subset of all profile changes, Kevin's 4% is the more conservative and internally consistent number. Per team convention, we use Kevin's figure when estimates conflict.

**Step 4 — Calculate Daily Calls at Steady State**

$$\text{Steady-state daily calls} = \text{DAU} \times 7\%$$

| Scope | DAU | Daily Calls |
|-------|----:|------------:|
| Global | 18,000,000 | **1,260,000** |
| USA | 2,800,000 | 196,000 |
| CAN | 450,000 | 31,500 |
| AUS | 400,000 | 28,000 |
| **Test regions** (USA+CAN+AUS) | 3,650,000 | **255,500** |

**Step 5 — Calculate Monthly Calls**

$$\text{Monthly calls} = \text{Daily calls} \times 30$$

| Scope | Daily Calls | Monthly Calls |
|-------|------------:|--------------:|
| Global | 1,260,000 | **37,800,000** |
| Test regions | 255,500 | **7,665,000** |

---

## 3. Serving Scenarios

There are two serving options for Photo Review. They differ in *when* the LLM call happens and *how much* it costs.

### Option 1: Real-Time Review Generation

The LLM is called the moment a user opens the app (if they are eligible). The feedback is generated on the spot and shown to the user immediately.

[!NOTE] latency is ~1 min for now. "at the same day" would be more appropriate.
[!NOTE] Add details for the API pricing, caching for system prompt is available but it is quite small.
- **Latency**: User sees feedback within seconds of opening the app.
- **API pricing**: Standard (non-batch) rate.
- **Cost per request**: $0.0086

[!NOTE] this "when to choose this" section is not required.
**When to choose this**: If the product requires instant feedback on first open, or if the user population is small enough that real-time cost is acceptable.

### Option 2: Batched (24-Hour) Review Generation

[!NOTE] it could be multiple batch job.
Eligible users are collected throughout the day. Once per day (e.g., at midnight), all pending LLM calls are submitted as a single batch job. Results are stored and served from cache when the user next opens the app.

[!NOTE] no explicit time (1pm) decided.
- **Latency**: Up to 24 hours delay. A user who changes their profile at 1pm may not see updated feedback until the next morning.
- **API pricing**: Gemini Batch API at **50% discount**.
- **Cost per request**: $0.0086 × 50% = **$0.0043**

**When to choose this**: If the product can tolerate a delay (up to 24 hours) between profile change and feedback update, and cost savings are a priority.

### Cost Comparison

[!NOTE] Use Kilo/Mega notation. (e.g., 18M, 50K)

#### Day-1 (Cold Start) — One-Time Cost

| Scope | Calls | Real-Time Cost | Batch Cost |
|-------|------:|---------------:|-----------:|
| Global | 18,000,000 | **$154,800** | **$77,400** |
| Test regions | 3,650,000 | **$31,390** | **$15,695** |

#### Steady-State — Monthly Cost

| Scope | Monthly Calls | Real-Time ($/month) | Batch ($/month) |
|-------|-------------:|--------------------:|----------------:|
| Global | 37,800,000 | **$325,080** | **$162,540** |
| Test regions | 7,665,000 | **$65,919** | **$32,960** |

#### Budget Comparison

The allocated budget is **$50,000/month**.

| Scenario | Monthly Cost | Within Budget? |
|----------|------------:|:--------------:|
| Test regions, Batch | $32,960 | Yes |
| Test regions, Real-time | $65,919 | No |
| Global, Batch | $162,540 | No |
| Global, Real-time | $325,080 | No |

---

## 4. Assumptions

### 4.1 Maximum One Call per User per Day

We assume each user triggers at most **1 LLM call per day**, regardless of how many times they open the app or change their profile. This avoids runaway costs from heavy users.

- **Number**: Max 1 call/user/day
- **Source**: Kevin's traffic model and Ben's recommendation ("Batch per user, not per app open")

### 4.2 Profile Change Rate

We assume **~4% of DAU** makes at least one profile change per day. A "profile change" includes any of: add/delete photo, change bio, change prompt, add/delete interest, add/delete descriptor.

- **Number**: 4% of DAU/day
- **Source**: Kevin's analysis of a 1,000-user sample
- **Note**: Ben's notebook reports ~8.5% for photo changes specifically. We use Kevin's more conservative 4% for all profile changes.

### 4.3 Steady-State Rate

The total steady-state trigger rate is **~7% of DAU/day**, combining ~3% new/returning logins (first-time scores) and ~4% profile changes.

- **Number**: 7% of DAU/day
- **Source**: Kevin's ramp model ("stable state would be 7%")
- **Note**: There is minor overlap (a new login who also changes their profile), making 7% a slight overestimate.

### 4.4 Cold-Start Ramp-Down

After launch, it takes approximately **3 weeks** for the daily call volume to ramp down from 100% of DAU to the 7% steady state.

- **Number**: ~21 days ramp-down
- **Source**: Kevin's Mode report ("take about 3 weeks to reach a steady state of 7%")

### 4.5 Batch API Discount

Gemini's Batch API offers a **50% discount** on per-token pricing compared to standard (real-time) API calls.

- **Number**: 50% cost reduction
- **Source**: Gemini API pricing documentation

### 4.6 Eligible User Filtering (Not Applied)

Kevin noted: "We would also probably exclude some users who didn't meet the minimum profile criteria or already had good profiles." This would reduce call volume, but we do **not** apply this filter in the current estimate. If applied, steady-state costs would be lower than shown above.

### 4.7 All Profile Changes Are Effective (Conservative)
[!NOTE] Not all profile changes are effective in this estimation. Only photo, bio, prompt, interest, and descriptor.
We assume every profile change triggers a re-evaluation. In practice, some changes (e.g., name change) may not affect Photo Review output if the ML model does not use that field. Differentiating "effective" vs. "non-effective" changes could reduce call volume, but we do not apply this optimization in the current estimate.
