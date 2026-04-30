# #mgai-aura Slack — Photo Review Summary (Apr 16–30, 2026)

**Source:** Slack #mgai-aura (C06FB511ZRR), searched via slack_search_public_and_private
**Captured:** 2026-04-30

---

## Biggie Choi

- Prompt v4.6.x: added one-cluster-at-a-time UI logic; fixed replace/swap action to target only non-representative photos in repetitive cluster (action MUST use `photo_position` targeting non-representative photos when `repetitive_photos` is non-empty)
- Reviewing rSRR Predictor v1 results (regression, CBM learning, SmartPhoto alignment) — 4 sections: rsrr regression performance, comparison with profile-ranker task, CBM learning performance, SmartPhoto alignment
- Concept KB work: dropped t2/t3/t5 naming; moved toward demo_driven (bio/interest), diversity, general (3 categories)
- Confirmed: no-photo cases in 5000-set — 31 no-photo runtime, 0 labeled despite no-photo → data quality OK

## Claude Yoon

- T5 Photo-Coachable Concepts KB report v2.0 (PR #260 in matchgroup-ai/aura-demo): 530K rSRR Predictor v1 dataset, NYC 28,687 users, 25 concepts, Cohen's d, Bonferroni correction, gender × age stratified
- Training/inference code in ldm-models: PR #38 (training + causal inference), PR #43 (modularized inference on top of #38)
- Demo → ML logic doc: 28 coaching items (explicit + implicit logic); system generates exactly 28 coaching items
- Concept labeling dataset: concept_labeled_20260416, ~5,000 training points; evaluation 160 points at photo_coaching_160_v6
- LLM-based concept description verification: 10 personas each extracting meaning from concept names; cross-validate for confidence level (highest/high/decent/should-modify)

## Ken Lee

- Face close ratio data: identified tinder_events_delta.media_yolo table; photo-level, key columns: photo_id, is_processed_by_yolo, is_face_detected, face_close_ratio. Shared for potential use in AURA demo
- rSRR Predictor v1 training report plan shared with team

## Parker Cho

- Provided eval session URLs for Eval 4
- Directed Ken to use Prompt Factory Human Eval page (ai-demo.dev.matchgroupcentral.net/promptfactory/aura-evals) for Diversity Poses fail case review

## Juniper Han

- Eval 4 feedback Notion doc published: repetitive detection accuracy is comparatively low in Pro; Pro missing photos that should be caught, or catching photos that shouldn't be
- New product requirement: multiple repetitive clusters must be handled one at a time
- Sharing photo review demo links for Eval 5 (Flash-Lite with KB v1.0 prompt v4.6.1)
- Final Photo Review Figma design fixed; requesting daily meeting walkthrough (~5 min)

## Link Lee

- Completed LP-570 (done)
- Submitted PR #69 to prompt-factory: Offloading experiment (small scope, included lesson-learned doc); flagged CRS as busy, deferred meeting
- Provided rsrr-predictor/v1 data path: /data/mgai/aura/rsrr-predictor/v1
- Asked Ken and Claude about AURA training data (user images) for CRS face detection dataset
- Transferred to Tailor (Chemistry) project ~Apr 21

## Owen Lee (cross-team)

- Question in #mgai-aura: should SOUP review "Option 2 Kick-off Decision Brief" or "Backend Contract"?

## Harriet Son (PM Manager, in #mgai-pm-tl)

- After Shurain 1:1: Shurain may be more interested in Photo Review demo than Prompt Factory; want to show examples from both at show and tell
