# Photo Review — LLM Cost Estimation

> **Sources**:
> - `00_note/2026-04-10-photo-review-llm-cost-estimation.md` (draft outline with reviewer notes)
> - `00_note/2026-04-10-photo-review-kevin-trideep-discussion-traffic.md` (Kevin's traffic model & Slack discussion)
> - `00_note/2026-04-10-photo-review-ben-dbx-notebook.md` (Ben's Databricks notebook: API volume & cost)
> - `01_work/2026-04-09-llm-cost-crosscheck-v1.md` (cross-check analysis, LP-485)
> - AURA Cost Dashboard (referenced, not directly accessed)

---

## TL;DR

At global steady state, Photo Review costs roughly $163K/month with batch API or $325K/month with real-time API, assuming Gemini 3.1 Flash-Lite at ~$8.6K per million requests. Day-1 cold start adds a one-time spike of ~$77K (batch) or ~$155K (real-time).
Using the batch serving option keeps the monthly cost within the $50K budget only if we scope to test regions first (USA + CAN + AUS steady state: ~$33K/month batch).

| Scope | Daily Calls (Steady) | Monthly Cost (Batch) | Monthly Cost (Real-time) |
|-------|---------------------:|---------------------:|-------------------------:|
| Test regions (USA+CAN+AUS) | 255.5K | **$33K** | $66K |
| Global | 1.26M | $163K | $325K |

## 1. LLM Cost per Request

We use Gemini 3.1 Flash-Lite Preview (`gemini/gemini-3.1-flash-lite-preview`) as the baseline model for all cost estimation below.

| #Requests | Cost |
|----------:|-----:|
| 1 | $0.0086 |
| 1M | $8,600 |

This per-request cost is an all-in number that accounts for the typical token usage of a Photo Review call (system prompt, user profile text, multi-image vision tokens, and output coaching text). See the [AURA Cost Dashboard](https://ai-demo.dev.matchgroupcentral.net/aurademo/demo/cost-dashboard) for the detailed breakdown of input/output token parameters or other models.

In practice, cost per request may vary with prompt length, number of photos, photo resolution, and output length. We treat these variations as negligible for planning purposes.

> Reference: [Gemini 3.1 Flash-Lite: Built for intelligence at scale](https://blog.google/innovation-and-ai/models-and-research/gemini-models/gemini-3-1-flash-lite/)

## 2. Traffic Quantities

### 2.1 DAU Baseline

| Country | DAU |
|---------|----:|
| Global | 18M |
| USA | 2.8M |
| CAN | 450K |
| AUS | 400K |
| English regions total (USA+CAN+AUS) | 3.65M |
Source: databricks `published_prod.analytics_rollups.app_open_first_last_uid_day`

### 2.2 When Does a User Trigger an LLM Call?

A user triggers exactly one Photo Review LLM call per day, and only if one of the following conditions is met:

1. **First login after launch (cold start)** — The user has never received a Photo Review score. The very first time they open the app after Photo Review is turned on, the system calls the LLM to generate feedback.
2. **Profile change** — After their initial score, a returning user triggers a new LLM call only when they change their profile (add/delete a photo, change bio, change prompt, add/delete an interest or descriptor).

If a user opens the app but has already been scored and has not changed their profile, no LLM call is made. There is a maximum of 1 call per user per day.

### 2.3 Step-by-Step: How Many Calls per Day?

**Step 1 — Day 1 (Cold Start)**

On the very first day Photo Review is turned on, nobody has been scored yet. So every user who opens the app that day will trigger one LLM call.

- Day-1 calls = DAU = 18M (global)

**Step 2 — Days 2 through ~21 (Ramp-Down)**

Starting on day 2, users who were already scored on day 1 do NOT trigger a new call (unless they changed their profile). Only two groups of users generate calls:

- Newly returning users who missed day 1 and open the app for the first time since launch.
- Profile changers who update their photos, bio, prompts, or interests.

Each day, the pool of "never-scored" users shrinks, so the number of first-time calls drops. Kevin's analysis shows this ramp-down takes about 3 weeks to reach steady state.

**Step 3 — Steady State (After ~3 Weeks)**

Once nearly all active users have been scored at least once, daily calls come from two sources only:

| Source | Rate (% of DAU) | Rationale |
|--------|----------------:|-----------|
| New/returning logins (first-time scores) | ~3% | New registrations + reactivated users who haven't been scored |
| Profile changes | ~4% | Kevin's sample of 1,000 users: ~4% make any profile change on a given day |
| Total steady-state rate | ~7% | Sum of above (with minor overlap, making this a slight overestimate) |

> Note on the 4% figure: Ben's notebook estimates 8.5% of DAU change photos daily, which is higher than Kevin's 4% for all profile changes combined. The two measurements likely differ in methodology or scope. In this document, we use Kevin's figure.

**Step 4 — Calculate Daily Calls at Steady State**

Steady-state daily calls = DAU x 7%

| Scope | DAU | Daily Calls |
|-------|----:|------------:|
| Global | 18M | 1.26M |
| USA | 2.8M | 196K |
| CAN | 450K | 31.5K |
| AUS | 400K | 28K |
| Test regions (USA+CAN+AUS) | 3.65M | 255.5K |

**Step 5 — Calculate Monthly Calls**

Monthly calls = Daily calls x 30

| Scope | Daily Calls | Monthly Calls |
|-------|------------:|--------------:|
| Global | 1.26M | 37.8M |
| Test regions | 255.5K | 7.67M |

## 3. Serving Scenarios

There are two serving options for Photo Review. They differ in when the LLM call happens and how much it costs.

### Option 1: Real-Time Review Generation

The LLM is called when a user opens the app (if they are eligible). The feedback is generated and shown to the user on the same day.

- Latency: ~1 minute per generation. Users see feedback on the same day.
- API pricing: Standard (non-batch) rate. Caching for system prompt is available but the cost reduction is small given the prompt size.
- Cost per request: $0.0086

### Option 2: Batched (24-Hour) Review Generation

Eligible users are collected throughout the day. Pending LLM calls are submitted as batch jobs (one or more per day). Results are stored and served from cache when the user next opens the app.

- Latency: Up to 24 hours delay between profile change and updated feedback.
- API pricing: Gemini Batch API at 50% discount.
- Cost per request: $0.0086 x 50% = $0.0043

### Cost Comparison

#### Day-1 (Cold Start) — One-Time Cost

| Scope | Calls | Real-Time Cost | Batch Cost |
|-------|------:|--------------:|-----------:|
| Global | 18M | $154.8K | $77.4K |
| Test regions | 3.65M | $31.4K | $15.7K |

#### Steady-State — Monthly Cost

| Scope | Monthly Calls | Real-Time ($/month) | Batch ($/month) |
|-------|-------------:|--------------------:|----------------:|
| Global | 37.8M | $325K | $163K |
| Test regions | 7.67M | $66K | $33K |

#### Budget Comparison

The allocated budget is $50K/month.

| Scenario | Monthly Cost | Within Budget? |
|----------|------------:|:--------------:|
| Test regions, Batch | $33K | Yes |
| Test regions, Real-time | $66K | No |
| Global, Batch | $163K | No |
| Global, Real-time | $325K | No |

Only the test-region + batch combination fits within the $50K/month budget. To operate globally within budget, either the model cost must decrease ~3x or the eligible user population must be further filtered.

## 4. Assumptions

### 4.1 Maximum One Call per User per Day

Each user triggers at most 1 LLM call per day, regardless of how many times they open the app or change their profile.

- Value: Max 1 call/user/day
- Source: Kevin's traffic model and Ben's recommendation ("Batch per user, not per app open")

### 4.2 Profile Change Rate

~4% of DAU makes at least one profile change per day. A "profile change" that triggers re-evaluation includes: add/delete photo, change bio, change prompt, add/delete interest, add/delete descriptor.

- Value: 4% of DAU/day
- Source: Kevin's analysis of a 1,000-user sample
- Note: Ben's notebook reports ~8.5% for photo changes specifically. We use Kevin's figure per team convention (R1).

### 4.3 Steady-State Rate

The total steady-state trigger rate is ~7% of DAU/day, combining ~3% new/returning logins (first-time scores) and ~4% profile changes.

- Value: 7% of DAU/day
- Source: Kevin's ramp model ("stable state would be 7%")
- Note: There is minor overlap (a new login who also changes their profile), making 7% a slight overestimate.

### 4.4 Cold-Start Ramp-Down

After launch, it takes approximately 3 weeks for the daily call volume to ramp down from 100% of DAU to the 7% steady state.

- Value: ~21 days ramp-down
- Source: Kevin's Mode report ("take about 3 weeks to reach a steady state of 7%")

### 4.5 Batch API Discount

Gemini's Batch API offers a 50% discount on per-token pricing compared to standard (real-time) API calls.

- Value: 50% cost reduction
- Source: Gemini API pricing documentation

### 4.6 Eligible User Filtering (Not Applied)

Kevin noted: "We would also probably exclude some users who didn't meet the minimum profile criteria or already had good profiles." This would reduce call volume, but we do not apply this filter in the current estimate. If applied, steady-state costs would be lower than shown above.

### 4.7 Effective Profile Change Scope

Not all possible profile changes trigger a re-evaluation. Only the following changes are considered "effective" and trigger an LLM call: add/delete photo, change bio, change prompt, add/delete interest, add/delete descriptor. Other changes (e.g., name change) do not trigger re-evaluation because the ML model does not use those fields.
