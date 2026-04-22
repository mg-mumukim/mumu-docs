| Key           | Value                                                                                              |
| ------------- | -------------------------------------------------------------------------------------------------- |
| Source        | `https://docs.google.com/document/d/1C7rteifFoWqU-qwCaBH7322S2zX3fcqB5B6ou-C6M0E/edit?usp=drivesdk` |
| Downloaded at | `2026-04-22 10:26:10`                                                                              |

---

# Photo Suppression V2 \-- Data Collection Experiment

# Photo Suppression V2 \-- Data Collection Experiment

## Resources

* Engineering: [Trideep Rath](mailto:trideep.rath@gotinder.com) (ML) [Taeyoung Sohn](mailto:taeyoung.sohn@gotinder.com) (MLSE) [Gabriela Rodriguez-Florido](mailto:gabriela.florido@gotinder.com) (Backend support)  
* Product:   
* Analytics: [Kevin Celustka](mailto:kevin.celustka@gotinder.com)

## Objective

Run an impression-level randomized experiment that suppresses a single photo per user to collect **per-photo marginal value data**.

### Direction 1: Better Product Understanding

This data lets us answer questions we currently cannot:

- **Value of photo count**: What is the rSRR drop when a user goes from 6 photos to 5? From 4 to 3? Does this differ by gender, market, or user quality?  
- **Value of photo position**: Is the 2nd photo more valuable than the 5th? How much of a profile's value is concentrated in the top 3 vs. evenly spread?  
- **Value of photo content**: At a content level, what types of photos (group shots, outdoor, selfies, etc.) contribute the most marginal value? Are there photo types that are redundant or even negative-value?  
- **SmartPhotos-era baselines**: The 2020 suppression test found diminishing returns beyond \~5 photos (women) / \~4 photos (men). Do these thresholds still hold now that SmartPhotos reorders profiles?

### Direction 2: Features We Can Build

The per-photo value signal directly powers new product capabilities:

- **Photo health score**: Tell users "This photo is adding X% to your profile" or "This photo may not be helping" \-- enabling data-driven photo coaching  
- **Negative-value photo detection**: Identify photos that, when removed, actually *improve* outcomes (e.g., redundant photos, low-quality shots that drag down profile perception). Could power nudges like "Consider replacing this photo"  
- **SmartPhotos ranking improvements**: Use predicted per-photo marginal value as an additional ranking signal, beyond just first-photo optimization  
- **Photo count guidance**: Personalized recommendations for how many photos a user should have, based on where diminishing returns kick in for their profile type

## Proposed Experiment Design

### Randomization

Two levels of randomization:

**Level 1 — Swipee-level (who is in the experiment):**

- **Control (X%)**: User is not part of the experiment. All photos shown normally at every impression.  
- **Variant (100-X%)**: User is enrolled in the experiment. One photo is randomly selected as the suppression candidate and **fixed for the duration of the experiment**.

X controls what percentage of users are affected. Since suppression can degrade the user experience, keeping X high (e.g., 95% control) limits the number of impacted users — see Open Questions.

**Level 2 — Impression-level (enables within-user comparison):** For each variant user, at every impression (swiper-swipee pair), randomize 50/50:

- **Control (Show all photos)**: Full profile in SmartPhotos order (N photos)  
- **Variant (Suppress)**: Profile shown without the pre-selected photo, remaining photos in SmartPhotos order (N-1 photos)

This gives us a clean **within-user paired comparison** — the same user is seen with and without a specific photo, removing all user-level confounders (attractiveness, profile quality, etc.). The rSRR difference between the two impression types is the **causal marginal value** of that photo.

**Implementation details:**

- **Swipee assignment**: Done via Phoenix at the user level.  
- **Photo selection**: For variant users, compute `photo_hash = hash(concat(photo_ids in SmartPhotos order)) % photo_count`. This deterministically selects which photo to suppress based on the current photo set — no state needed, consistent across impressions.  
- **Photo additions/removals**: If a user changes their photo set, the `photo_hash` naturally re-evaluates against the new set, so the suppressed photo may change. We log `photo_hash` and `suppressed_photo_id` at every impression, so during analysis we group by (user, photo\_hash) for clean per-photo comparisons even when the photo set shifts mid-experiment.

### Eligibility & Targeting

- **Swipee photo count**: Must have **3+ photos** (open question: should this be higher?)  
- **Subscriber eligibility**: TBD (2020 test excluded subscribers on both sides)

### Market & Duration

- **Market**: TBD (2020 test was global; touchup data collection ran in CHL)  
- **Duration**: TBD (2020 test ran \~1 week; touchup data collection ran 4 weeks)

### Event Logging

For all recs in the experiment, log the following fields in recs.rate (or equivalent):

**New fields (need to be added):**

| Field | Type | Description |
| :---- | :---- | :---- |
| `suppressed_photo_id` | string | ID of the photo selected for suppression (logged in both control and treatment for analysis) |
| `photo_hash` | string | `hash(concat(photo_ids in SmartPhotos order)) % photo_count`. Enables grouping by (user, photo\_hash) to handle mid-experiment photo changes |
| `suppressed_photo_position` | int | SmartPhotos position of the suppressed photo (0-indexed) |
| `is_suppressed` | bool | Whether the photo was actually hidden this impression |
| `shown_photo_ids` | list\[string\] | Ordered list of photo IDs actually shown to the swiper (post-suppression, in SmartPhotos order) |
| `swipee_media_count_shown` | int | Media count actually shown (media\_count \- 1 for treatment) |
| `is_eligible` |  |  |

**Existing fields (already in recs.rate or joinable):**

| Field | Source | Description |
| :---- | :---- | :---- |
| `experiment_bucket` | Phoenix tables | Control or treatment assignment |
| `swipee_media_count` | recs.rate | Total media count before suppression |
| `swipee_photo_ids` | recs.rate | Full ordered list of photo IDs (pre-suppression) |
| `like` | recs.rate | Whether the swiper liked the swipee |

### Primary KPI

- **rSRR** (received swipe-right rate) at the impression level  
  - Delta: rSRR(all photos) \- rSRR(photo suppressed) for the same user

## Open Questions

1. **Minimum photo count**: Should eligibility be 3+ photos or higher? Lower threshold \= more data but higher risk of degrading thin profiles.  
2. **Subscriber eligibility**: Should subscribers be included? Excluding them reduces risk but limits generalizability (2020 excluded subscribers).  
3. **Market scope**: Single market (like CHL for touchup) or broader? Single market limits risk but may not generalize.  
4. **Which photo to suppress**: Pure random selection, or stratified by position (ensure we get enough first-photo suppressions for position analysis)?  
5. **Interaction with other experiments**: Need to check for conflicts with the photo touchup experiment and any active SmartPhotos tests.

## Prior Work: 2020 Photo Suppression Test

**The 2020 Photo Suppression Test** ([BA readout](https://gotinder.atlassian.net/wiki/spaces/DATA/pages/26276927), [Mode report](https://app.mode.com/tinder/reports/836230cca3a1)) established foundational learnings:

- Suppression hurts rSRR, first photo matters most, diminishing returns beyond \~5 photos (women) / \~4 photos (men)

**What this experiment adds beyond the 2020 test:**

- **Per-photo content-level signal** \-- the 2020 test analyzed by position (first/last/random) but never tied impact back to *what was in the photo*  
- **Training data for ML models** \-- no reusable dataset linking photo features to marginal value  
- **SmartPhotos-era estimates** \-- the 2020 test excluded SmartPhotos users; today SmartPhotos is the default ranking, so those estimates may not hold  
- **Identification of negative-value photos** \-- the 2020 test only showed suppression hurts in aggregate; we don't know if there are individual photos whose removal actually *helps*

*Action Items:*

- *[Kevin Celustka](mailto:kevin.celustka@gotinder.com)How many impressions do we need to get stat sig difference? If we need 10k \- 100k such users, what should be the market size.* 

## References

- [2020 Photo Suppression Test \- BA Readout](https://gotinder.atlassian.net/wiki/spaces/DATA/pages/26276927)  
- [2020 Photo Suppression Test \- Mode Report](https://app.mode.com/tinder/reports/836230cca3a1)  
- [2020 Photo Suppression Test \- Product Spec](https://gotinder.atlassian.net/wiki/spaces/PRODUCT/pages/30899434)  
- [2020 Photo Suppression Test \- Backend Design](https://gotinder.atlassian.net/wiki/spaces/UG/pages/33359830)  
- [Photo Touchup ML Data Collection Experiment](https://docs.google.com/document/d/1Mq1TjKiim1pww5E2usTNpB6dlyxXZgmRTOeXcwB6JHY)  
- [Photo Suppression Test Writeup (Glean)](https://docs.google.com/document/d/1tsp0OaMifef3LOSI483RcpfGJzv4A3AYUxtjN5OBaDg)

Notes:

- Experimentation setup: Should be straightforward  
- Event:   
  - Investigation ?   
  - Solution would be in simlar lines to SpV5  
- Randomization  
  - Media modification logic changes  
  - No api changes  
  - Hashing logic  
  - Photo count filtering should be relatively straightforward  
- Testing in dev  
  - Recs Viewer and app parity  
  - Logging

# Meeting notes
