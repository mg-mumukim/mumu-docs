> Sources:
> - note/source/2026-04-22-photo-suppression-v2-data-collection-experiment-gdocs.md
> - https://docs.google.com/document/d/1C7rteifFoWqU-qwCaBH7322S2zX3fcqB5B6ou-C6M0E/edit?usp=drivesdk

---

# Photo Suppression V2 — Design Review

## What This Experiment Is

The goal of this experiment is to measure the per-photo marginal value: how much does each photo contribute to a user's swipe-right rate (rSRR)?

The design works in two layers. At the first layer (user level), a portion of swipees are enrolled in the experiment. Each enrolled user has one photo selected as the suppression candidate. This photo stays fixed for the entire experiment. At the second layer (impression level), each time an enrolled swipee is shown to a swiper, there is a 50/50 chance: either all photos are shown, or the selected photo is hidden. This within-user comparison removes individual-level confounders (e.g., overall attractiveness) and isolates the causal impact of the one photo.

The primary metric is rSRR measured at the impression level. The difference in rSRR between "all photos shown" impressions and "photo suppressed" impressions is treated as the causal marginal value of that photo.

The experiment builds on the 2020 Photo Suppression Test. It adds per-photo content signals, training data for ML models, and estimates that account for SmartPhotos — which was excluded from the 2020 test.

---

## Comment Summary

| Priority | Comment |
| -------- | ------- |
| Critical | Power analysis must come before Market, Duration, and X% decisions |
| High | Hash-based photo selection may produce uneven position coverage across users |
| Medium | Mid-experiment photo changes can introduce selection bias that logging alone does not fix |

---

## Comment 1 — Run the power analysis first

**Priority: Critical**

### The Issue

Three key design decisions are currently unresolved:

- Market scope: "TBD (2020 test was global; touchup data collection ran in CHL)"
- Duration: "TBD (2020 test ran ~1 week; touchup data collection ran 4 weeks)"
- Control ratio (X%): "X controls what percentage of users are affected"

The document lists a power analysis as a follow-up action item: "How many impressions do we need to get stat sig difference? If we need 10k - 100k such users, what should be the market size."

### Why It Matters

Market scope, duration, and control ratio are not independent choices. They are the answer to one question: can this experiment collect enough data to measure the effects we care about?

The logic works like this. The power analysis tells you how many impressions you need per (user, photo) group to detect a statistically significant difference in rSRR. Once you have that number, you work backwards:

- Duration: how many weeks does it take to collect that many impressions in a given market?
- Market scope: which market produces enough volume in that timeframe?
- X%: how many enrolled users does it take to reach the required impression count?

These three decisions cannot be made until the power analysis is done. Right now, the document leaves all three as TBD — which means no part of the experiment design can be finalized. The power analysis is a prerequisite, not a follow-up.

### What Needs to Change

Complete the power analysis before resolving Market, Duration, or X%. The output should specify the minimum number of impressions required per (user, photo_hash) group, and the sample size assumptions (effect size, significance level, power).

---

## Comment 2 — Hash-based selection may under-sample certain photo positions

**Priority: High**

### The Issue

The document describes this method for selecting which photo to suppress per user:

> "photo_hash = hash(concat(photo_ids in SmartPhotos order)) % photo_count"

The same document lists position analysis as one of the core research questions:

> "Is the 2nd photo more valuable than the 5th? How much of a profile's value is concentrated in the top 3 vs. evenly spread?"

It also asks in Open Questions:

> "Pure random selection, or stratified by position (ensure we get enough first-photo suppressions for position analysis)?"

### Why It Matters

For position analysis to be valid, suppressed photos must be distributed roughly evenly across positions (position 1, 2, 3, etc.) across the user population. If position 1 is rarely selected for suppression, you will not have enough data to measure the value of the first photo.

The hash method does not guarantee uniform position distribution. Whether the distribution is even depends on how photo IDs are assigned. If photo IDs are sequential (e.g., newer uploads get higher IDs), and SmartPhotos consistently places certain types of photos at certain positions, then the hash output will reflect that structure — and some positions will be selected more often than others.

The Open Question acknowledges this risk but the Implementation Details section already specifies the hash method without verifying whether it achieves uniform coverage. The two sections do not connect.

### What Needs to Change

Before finalizing the implementation, verify empirically that the hash method produces uniform distribution across photo positions in a sample of real user data. If it does not, add position stratification: assign each enrolled user to suppress a photo from a specific position, ensuring each position is covered in proportion to the analysis needs.

---

## Comment 3 — Photo changes during the experiment can bias the results

**Priority: Medium**

### The Issue

The document acknowledges that users may change their photos mid-experiment and proposes a solution:

> "If a user changes their photo set, the photo_hash naturally re-evaluates against the new set, so the suppressed photo may change. We log photo_hash and suppressed_photo_id at every impression, so during analysis we group by (user, photo_hash) for clean per-photo comparisons even when the photo set shifts mid-experiment."

### Why It Matters

The proposed solution addresses label consistency (which photo was suppressed). It does not address two deeper problems.

**First: thin data.** When a user changes photos, the new (user, photo_hash) group starts accumulating impressions from zero. If the change happens late in the experiment, the new group may not collect enough impressions to produce a reliable estimate. The document does not define a minimum impression threshold for a group to be included in the analysis.

**Second: selection bias.** Users do not change photos randomly. A user whose suppressed photo is their most effective one will see a drop in matches, and is more likely to notice and update their profile. When this happens, their pre-change (user, photo_hash) group remains in the analysis — but without the full treatment period — and their post-change behavior reflects a different photo set.

The result is that users who were most harmed by suppression are systematically more likely to exit the original suppression group. The analysis will be left with users who were less affected by suppression, which makes the measured impact of suppression look smaller than it actually is.

### What Needs to Change

Add a minimum impressions threshold per (user, photo_hash) group below which the group is excluded from the primary analysis. Additionally, treat users who change photos during the experiment as a separate analytical stratum and check whether their pre-change rSRR pattern differs from users who did not change photos. This will reveal whether selection bias is present and how large it is.
