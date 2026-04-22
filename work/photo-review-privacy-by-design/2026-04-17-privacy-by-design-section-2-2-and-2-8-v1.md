# Privacy by Design - Section 2.2 and 2.8 Combined Draft

Sources:
- `note/source/2026-04-03-privacy-by-design-system-context.md`
- `note/task/2026-04-03-privacy-by-design-aura-demo-request.md`
- `note/task/2026-04-14-privacy-by-design-qwen-3-vl-request.md`
- `note/source/2026-04-10-profle-ranker-h100-benchmark.md`
- HuggingFace model cards: Qwen VL series (license: Apache 2.0)
- GitHub py-why/EconML (MIT), cdt15/lingam (MIT), dmlc/xgboost (Apache 2.0), microsoft/LightGBM (MIT)
- Slack #mgai-aura, #smart-photos-v4, #ml-photo-review threads
- GitHub TinderBackend/revive.mlinfra SmartPhotos vLLM Runbook
- Prior draft: Section 2.2 v7 (2026-04-08)
- Prior draft: Section 2.8 v1 (2026-04-09)

---

**Changes from Section 2.2 v7 and Section 2.8 v1**

- Added "Model Pipeline" subsection to Step 3 (Section 2.2) with an explicit input/output mapping for each model stage: concept discovery, concept prediction (Qwen VL), and causal inference (GBDT).
- Added an intermediate artifact note in Step 3 identifying user-profile/concept-score as derived Personal Information and documenting the consent disclosure gap.
- Added HyperPod (intermediate ML artifacts) row to the retention summary table in Section 2.8.
- Added new subsection "HyperPod -- Intermediate ML Artifacts" in Section 2.8 covering the retention policy and consent disclosure gap for user-profile/concept-score.

---

## 2.2 How will Personal Information be processed?

Response:

**Executive Summary**

The AURA project uses existing Tinder user data (demographics, profile content, photos, bio text, and behavioral metrics -- excluding direct identifiers) to generate personalized profile coaching. Data is sourced from the Tinder Databricks warehouse, scoped to AUS/NZL/CAN/US only, explicitly excluding EU/EEA/Switzerland/US-Illinois.
Data and model usages can be categorized as 4 different pillars.

- **Research**: AI agents run aggregate statistical analyses to build a "Knowledge Base". All outputs are aggregate-level and contain no individually identifiable data.
- **Model training**: Custom models discover domain concepts, predict their presence per user, and estimate causal treatment effects.
- **Demo**: Custom model + Gemini API generate natural language coaching for individual users. Photos are passed unblurred (required for expression/composition evaluation); no facial identification or biometric extraction is performed. Bio text may contain sensitive information, but coaching topics are pre-fixed to an approved non-sensitive concept list.
- **Evaluation**: Human and/or LLM-as-a-judge continuously monitors output quality and guardrail compliance -- bio safety, photo identification incidents, and sensitive concept surfacing.

Concept governance: concepts posing legal extraction risk are removed entirely; those safe to extract but inappropriate to surface are retained for model accuracy only (coachable = N). All concept lists and Knowledge Base entries are human-reviewed before deployment or export.

Extracted data is retained for no more than 3 months under Match Group's DLP. Third-party LLM vendors (Anthropic, OpenAI, Google) are used within MG-approved environments; only Gemini is used for the user-facing demo.

---

The data flow consists of the following stages:

### Step 1: Data Collection from Existing Sources

User data is sourced from Tinder's existing Databricks infrastructure. No new data collection mechanisms are introduced -- only data already available through Match Group's existing data infrastructure is accessed. The following categories of data are used:

#### 1a. Demographics (excluding user identifiers)

User identifiers such as uid, user number, name, and email address are explicitly excluded from all processing.

| Field | Example Values | Purpose |
|-------|---------------|---------|
| Age | numeric | Cohort differentiation |
| Gender | male / female / non-binary | Cohort differentiation |
| Target gender | male / female / non-binary | Cohort differentiation |
| Country code | US, AU, NZ, CA | Cohort differentiation |
| City | New York | Cohort differentiation |
| Language | English | Language filtering |

Demographic features are used to differentiate appropriate treatment for the user, because effective and acceptable advice varies across cohorts. For example, certain photo attributes (e.g., height visibility) show different effects on dating outcomes between different genders.

#### 1b. User-selected profile content

| Sub-category | Example Values |
|-------------|---------------|
| Job titles, employers | Software Engineer, Google |
| School | NYU |
| Lifestyle preferences | Drinking, smoking, workout, dietary preference |
| Basics | Zodiac, education level, family plan, communication style |
| Interests | Travel, hiking, coffee |
| Relationship intent | Long-term, short-term |

#### 1c. User-generated content (UGC)

| Type | Format | Description |
|------|--------|-------------|
| Bio | Text | Free-text profile description written by the user |
| Photos | Image | Profile photos uploaded by the user (up to 9) |

User-selected and user-generated content together enable personalized coaching. For example, the LLM may recommend "Add a hiking photo" when: (1) the user has selected "Hiking" as an interest but has not uploaded a hiking-related photo, and (2) uploading hiking photos has been statistically shown to improve outcomes for the cohort to which the user belongs. Those statistically valid treatment candidates are curated and injected from research.

#### 1d. User behavior data

| Field | Description |
|-------|-------------|
| Swipe history | Record of user's like/pass decisions |
| SmartPhoto score | Tinder's internal model output to rank photos |
| Impression count | Number of times each photo has been shown to other users |

User behavior data is used to infer causal relationships between "Dating Concepts" (profile elements such as photo attributes or bio content) and "Dating Outcomes" (metrics such as rSRR). These relationships are required to construct the "Knowledge Base", which is fixed after human review.

#### 1e. Technical flags

| Flag | Description |
|------|-------------|
| SmartPhoto enabled | Whether the automatic photo ordering feature is active |
| Selfie verified | Whether the user has completed selfie verification |

These are used as control variables in research analysis and model training.

#### Data Management

- **Source governance**: All source data is managed by Match Group's Data Lifecycle Policy (DLP). The project accesses only currently available (non-deleted, DLP-compliant) data.
- **Geographic scope**: English-speaking geos only -- AUS, NZL, CAN, and US. Data from EU/EEA/Switzerland/US-Illinois is explicitly excluded.
- **Prior usage**: The March 2026 demo and preliminary research were conducted with New York City data only.
- **Retention**: See Section 2.8 for data retention and deletion policies.

#### Note on Biometric Data

The LLMs are not explicitly prompted to extract biometric data from user photos. Photo analysis identifies domain-level visual concepts (e.g., "outdoor setting", "group photo", "clear face shot") but does not perform facial recognition, facial geometry extraction (FaceMap), or 1:N/1:1 person identification. Identifying duplicate photos is permitted, but not by recognizing the person -- only by visual similarity of the image itself.

However, even without explicit prompting, LLMs may internally perform gray-zone operations as part of achieving their assigned task -- for example, implicitly processing facial features to assess photo composition, or inferring demographic attributes to contextualize a recommendation. To mitigate this risk, explicit guardrails are embedded in the system prompt that prohibit identity inferences, biometric analysis, and sensitive attribute extraction. These guardrails are continuously monitored through the evaluation process (Step 5), and prompts are iteratively refined when violations are detected.

### Step 2: Research -- Knowledge Base Generation

AI research agents query Tinder Databricks to validate human-defined hypotheses about profile elements and dating outcomes. The research process follows these steps:

1. **Cohort segmentation**: Users are grouped into cohorts based on demographic features (age, gender, target gender, geographic region) to control for population-level differences in dating behavior.
2. **Hypothesis formulation**: An engineer defines a research hypothesis -- e.g., "Do outdoor photos improve rSRR?" or "How fatal are sunglasses in profile photos?"
3. **Data querying**: The AI research agent translates the hypothesis into SQL queries and executes them against the Databricks warehouse, retrieving aggregate behavioral data (swipe history, rSRR) for the target cohort.
4. **LLM-assisted classification** (when applicable): For hypotheses requiring visual or textual understanding (e.g., "Does a cluttered background reduce rSRR?"), the LLM classifies profile elements of users in the cohort at scale. The classification results are joined with behavioral data for analysis.
5. **Statistical analysis**: The agent performs correlation and confounder handling between the classified profile attributes and rSRR outcomes within each cohort.
6. **Report generation**: A single structured report is produced per hypothesis, documenting findings with statistical rigor (p-values, effect sizes, sample size validation, confounder analysis).
7. **Human review and curation**: Engineers review each report for validity and legal compliance. The list of domain concepts is confirmed and fixed at this stage to exclude concepts with legal concerns. Validated insights are curated into the Knowledge Base and are not exported or consumed by downstream systems (model training, demo) until review is complete.

- **Human safeguard**: The engineer providing the prompt is responsible for restricting the agent's search space to prevent extraction of sensitive social attributes.
- **Output**: "Knowledge Base" -- cumulative statistical insights about which profile elements (e.g., outdoor photos, clear face shots) contribute to positive dating outcomes (e.g., higher rSRR). All Knowledge Base outputs are aggregate-level statistics (cohort-level averages, effect sizes, conditional distributions) and do not constitute Personal Information -- no individual user's data is identifiable in the final output.
- **LLM vendors**: See Step 6.

### Step 3: Custom Model Training

The same source data is used to train custom ML models with three objectives:

1. **Concept discovery**: Build a comprehensive list of domain concepts (e.g., "has curly hair", "outdoor setting", "group photo") by analyzing profile elements across a large population of users.
2. **Concept prediction**: For a given user's profile elements, predict which domain concepts are present in their photos and bio.
3. **Causal inference**: Calculate the conditional average treatment effect (CATE) of adjusting specific profile elements to determine the most impactful coaching advice for each user.

#### Model Pipeline

The three objectives map to a sequential model pipeline. Concept discovery and concept prediction share the same source data as input. The output of concept prediction (user-profile/concept-score) is used as the input to the causal inference model.

| Stage | Model | Input | Output |
|-------|-------|-------|--------|
| Concept discovery | Multiple approaches (research analysis, Qwen VL series, domain expertise) | User profile data and behavior data | Concept list (natural language) |
| Concept prediction | Qwen VL (custom) | User profile data and behavior data | User-profile/concept-score |
| Causal inference | GBDT-based custom model | User-profile/concept-score | Objective-related score (e.g., uplift) |

**Intermediate artifact -- user-profile/concept-score**: The concept prediction stage produces a derived dataset in which each user is associated with a predicted score per domain concept. This dataset is stored on HyperPod and is used as the exclusive input to the causal inference model for training. It currently follows the same 3-month retention cycle as the source dataset. The specific derived attributes stored in this dataset -- which domain concepts are predicted as present per user -- represent processed Personal Information inferred from raw user data. This derived data category has not been explicitly disclosed in current user consent terms. See Section 2.8 for retention details and the documentation of this gap.

Similar to Step 2, the model training process leverages the full range of source data described in Step 1.
Demographic features (age, gender, target gender, geographic region) serve as conditioning variables in the causal inference process, because the contribution of a given profile attribute to rSRR varies significantly across cohorts -- for example, the effect of adding an outdoor activity photo differs between genders and age groups.
User behavior data (swipe history, rSRR per photo) provides the outcome signal for estimating treatment effects. The final output of the trained models is a ranked list of actionable profile attributes (i.e., attributes the user can realistically change) and their estimated importance scores per cohort -- for example, "adding an outdoor activity photo" may rank as a high-impact treatment for a specific demographic segment.

The domain concept list undergoes the same human review process described in Step 2 before deployment. These models use distinct processing and storage locations (AURA demo) from the Research Agent (during research).

#### Model Details

Two categories of open-source software are used in Step 3, all under permissive open-source licenses (Apache 2.0 or MIT) that permit commercial use, modification, and distribution.

**Concept extraction (objectives 1 and 2)** uses the **Qwen VL (Vision-Language) model family** (Apache 2.0), a series of open-weight models developed by the Qwen team (Alibaba Cloud). Multiple model variants within this family (differing in parameter size) are evaluated during development; the final production variant is selected based on accuracy and infrastructure constraints. The Qwen VL architecture provides vision-language capability -- the ability to process both images and text jointly -- which is required for concept discovery and prediction, as these tasks must analyze profile photos alongside textual profile elements.

**Causal inference (objective 3)** uses the following open-source libraries. None of these are pre-trained models -- they are trained entirely from scratch on project-specific data. They operate on structured numerical data -- specifically, the concept scores produced by the Qwen VL model and the behavioral outcome data described in Step 1 -- and do not process raw photos or text directly.

| Library | License | Developer |
|---------|---------|-----------|
| EconML | MIT | Microsoft (py-why) |
| LiNGAM | MIT | cdt15 |
| XGBoost | Apache 2.0 | DMLC |
| LightGBM | MIT | Microsoft |

**Infrastructure and data handling**: All model training and inference are performed on HyperPod, Hyperconnect's internal GPU cluster. Unlike the third-party LLM API vendors listed in Step 6 (Anthropic, OpenAI, Google), these models operate entirely within the organization's own infrastructure. No user data is transmitted to external vendors for model training or inference. Data retention on HyperPod follows the same 3-month policy described in Section 2.8.

**Prior deployment within the organization**: An earlier generation of the Qwen VL family (Qwen2.5-VL) is currently deployed in production for Tinder's SmartPhoto v4 feature, establishing an operational precedent for this model architecture.

### Step 4: Demo -- Profile Coaching Generation

For individual users, the system generates personalized profile coaching:

1. **Custom model** identifies what domain concepts are present in the user's profile and where coaching opportunities exist, then ranks possible treatments in text format. The concept extraction may internally detect sensitive attributes. The curated list of concepts eligible for outbound coaching is currently being finalized to ensure only appropriate, non-sensitive topics are included.
2. **LLM API (Gemini)** generates a natural language coaching recommendation based on the model's output, following brand voice guidelines and guardrails (no identity inferences, no aggressive appearance evaluation, no biometric analysis).
3. **Output**: The user receives only the generated coaching text (e.g., "Add a hiking photo since you mentioned hiking as an interest"). Raw data, literal concepts, scores, and intermediate analysis are not exposed to the user.

**Sensitive data handling in LLM input**:

- **Bio text**: The user's bio is passed to the LLM as context. Because bio text is free-form, it may contain any type of sensitive information (e.g., health conditions, political views, religious beliefs). However, the set of coachable topics is pre-defined and fixed -- the LLM is constrained to generate advice only for approved concept categories. The LLM is therefore not expected to reference, leverage, or surface sensitive bio content in its output. Compliance with this constraint is continuously validated through the evaluation process (Step 5), with the system prompt iteratively refined based on findings.
- **Profile photos**: Photos are passed to the LLM without blurring or anonymization, because the coaching task requires evaluating facial expressions, photo composition, lighting quality, and overall visual impression -- all of which would be degraded or lost by such processing. As a result, photos may contain the user's identifiable face. As noted in Step 1 (Note on Biometric Data), no facial identification or verification is performed. Should such behavior occur as an unintended side effect, the evaluation process (Step 5) is designed to detect and flag these cases for prompt correction.

**Concept lifecycle: extraction, curation, and coaching eligibility**:

Domain concepts are the atomic building blocks of the coaching system. Each concept represents a discrete, observable profile attribute (e.g., `smiling_expression`, `outdoor_activity_signal`, `curly_hair_feature`). The concept list is constructed through three complementary sources: (1) research agent analysis that statistically validates causal relationships between profile attributes and dating outcomes, (2) literature review and domain expert intuition that proposes candidate concepts, and (3) custom model training followed by interpretation of learned internal features. In all cases, candidate concepts are verified for causal impact and the final list is confirmed through human review.

Once established, each concept is classified by coaching eligibility:

- **Coachable (Y)**: Attributes the user can realistically act on. For example, `smiling_expression`, `outdoor_activity_signal`, `group_photo_presence`, `clear_face_visibility`, and `blur` are all coachable -- users can add a smiling photo, take an outdoor shot, or replace a blurry image.
- **Not coachable (N)**: Attributes that are immutable personal traits (e.g., `curly_hair_feature`, `glasses_presence`), or that are sensitive or misaligned with product goals (e.g., `overweight_body_type`, `skin_complexion_quality`, `cleavage_emphasis`, `shirtless_photo_presence`). These concepts remain in the list because they contribute to the accuracy of the causal inference model -- omitting them would introduce confounding -- but they are never surfaced as coaching recommendations to the user.

Two distinct policies govern concept restriction:

1. **Removal from list**: If extracting a concept itself poses legal or regulatory risk (e.g., inferring sexual orientation, racial identity, or medical conditions), the concept is excluded from the list entirely and is not processed at any stage.
2. **Coachable = N**: If a concept can be safely extracted but should not be surfaced to the user -- either because coaching on it carries reputational risk (e.g., `skin_complexion_quality`), involves immutable traits (e.g., `blonde_hair_feature`), or does not align with product values (e.g., `luxury_affluence_signal`) -- the concept is retained for internal model accuracy but flagged as non-coachable. The LLM never generates recommendations based on these concepts.

### Step 5: Evaluation -- Quality Assurance and Guardrail Monitoring

Human and/or LLM-as-a-judge evaluates the generated coaching against the following criteria:

- Accuracy (correctly identifies profile gaps; does not infer beyond available information)
- Framing (follows content strategy guidelines)
- Visual proof (supported by actual profile photo state)
- Title/Benefit cohesion (logical connection between insight and stated benefit)
- Appropriateness (recommended actions are suitable and not offensive)
- Ease (recommended actions are realistic for the user to execute)
- Title/Action cohesion (action follows from identified insight)
- Diversity score (correct Low/Medium/High classification of the photo set)

Beyond product quality, the evaluation process also serves as continuous monitoring for privacy guardrail compliance. Specifically, it is designed to detect:

- **Bio safety violations**: Cases where the LLM references or surfaces sensitive information from the user's free-form bio text (e.g., health conditions, political views, religious beliefs) outside the pre-approved set of coachable topics.
- **Photo identification incidents**: Cases where the LLM performs or implies facial identification (1:N) or identity verification (1:1) as an unintended side effect of photo analysis.
- **Sensitive concept surfacing**: Cases where coaching output contains recommendations based on non-coachable concepts (e.g., appearance-based attributes marked as N in the concept list).

When violations are detected, the system prompt and guardrails are iteratively refined to prevent recurrence.

**Input**: Prompt-output pairs from the Demo stage.
**Output**: Quality scores and guardrail compliance flags, used internally for iteration and improvement.

### Step 6: Third-Party LLM Vendors

The following third-party LLM API vendors are used across Research, Demo, and Evaluation:

| Vendor | Product | Usage |
|--------|---------|-------|
| Anthropic | Claude API | Research, Evaluation |
| OpenAI | ChatGPT API | Research, Evaluation |
| Google | Gemini API | Research, Demo, Evaluation |

### Step 7: Output and Access

| Output | Audience | Description |
|--------|----------|-------------|
| Knowledge Base | MGAI team (internal) | Statistical insights on profile elements and dating outcomes |
| Custom Model | MGAI team (internal) | Trained models to predict dating concepts and estimate dating outcomes |
| Profile coaching | Individual end user | Natural language recommendations for profile improvement |
| Evaluation results | MGAI team (internal) | Quality scores for coaching output iteration |

### Step 8: Data Retention and Deletion

- Source data is governed by Match Group's Data Lifecycle Policy (DLP).
- Downloaded user profile data for research or demo is removed within 3 months. Deactivated account data is flushed on the same 3-month cycle.
- No project-specific data is retained indefinitely.

---

## 2.8 How long will the data be retained before it is anonymized or deleted?

Response:

All source data is governed by Match Group's Data Lifecycle Policy (DLP). All environments apply a maximum 3-month retention period for Personal Information. The project operates across three environments with different retention mechanisms, summarized below and detailed per environment.

| Environment | Retention | Deletion | Automated? | Verification |
|-------------|-----------|----------|------------|-------------|
| HyperPod (model training) | 3 months / account event | DLP pipeline | Phase 1: manual; Phase 2: automated (under development) | S3 inventory snapshot |
| HyperPod (intermediate ML artifacts) | 3 months / account event | DLP pipeline | Phase 1: manual; Phase 2: automated (under development) | S3 inventory snapshot |
| Local machines (research) | 3 months | Engineer manual deletion | No | None |
| MG Core S3 (demo, evaluation) | 3 months / account event | Manual deletion | No (process to be defined) | To be defined |

Project outputs that contain only aggregate or non-individual data -- Knowledge Base (cohort-level statistics), trained model weights, evaluation quality scores -- are retained for the project duration and are not subject to the above deletion cycle.

### HyperPod -- Custom Model Training

Structured data (demographics, behavior, flags) is received via Delta Sharing from Tinder Databricks. Unstructured data (photos) is currently accessed via direct S3 API (Phase 1, interim) and will transition to Databricks External Volume access only (Phase 2).

- **Retention period**: Maximum 3 months from data import.
- **Account events**: Account deactivation, photo deletion, and content purge events are detected via Delta Sharing from Tinder event tables (`trust_case_review`, `profile_delete_photo`, `trust_account_status`). Target deletion SLA is 48 hours (baseline; to be refined after Phase 1 operational data).
- **Deletion process**: (1) event detection and registration in tracking table, (2) FSx cache and HyperPod local copy deletion, (3) S3 object deletion, (4) verification by comparing daily S3 inventory snapshots before and after deletion; failure triggers manual intervention alert.
- **Audit logging**: Physical deletion, pseudonymization mapping deletion, and cache load history are logged in separate audit tables, accessible only to infrastructure leads and managers.
- **Current status (Phase 1)**: Deletion is manual -- engineers delete data after experiments or at the 3-month mark. The automated DLP pipeline described above is under active development.
- **Target status (Phase 2)**: Fully automated pipeline with event-driven deletion, S3 inventory verification, path-violation file quarantine, and DLP-unapplied data monitoring. Direct S3 API access will be blocked.

### HyperPod -- Intermediate ML Artifacts

The concept prediction stage produces a derived dataset (user-profile/concept-score) in which each user record is associated with a predicted score for each domain concept. This dataset is stored on HyperPod and is used as input to the causal inference model.

- **Retention period**: Maximum 3 months, following the same cycle as the source dataset.
- **Deletion mechanism**: Follows the same Phase 1 / Phase 2 DLP pipeline described in the HyperPod Custom Model Training section above.
- **Current classification**: This dataset is currently treated as an intermediate training artifact, managed under the same lifecycle as the source data.
- **Consent disclosure gap**: The extraction and retention of user-profile/concept-score has not been explicitly disclosed in current user consent terms. Specifically, what is stored in this dataset -- which domain concepts are predicted as present per user -- constitutes a new category of processed Personal Information inferred from raw user data. This gap requires either explicit disclosure to users of what derived attributes are extracted and retained, or confirmation from Legal that existing consent terms cover this use, before production deployment.

### Local Machines -- Agentic Research

Engineers use local machines (MacBooks) for data analysis and AI agent-driven research that does not require GPU resources.

- **Retention period**: Maximum 3 months, aligned with project-wide policy.
- **Deletion mechanism**: Manual deletion by the responsible engineer. No automated enforcement exists.

### MG Core S3 -- AURA Demo and Evaluation

Demo and evaluation services store copied user profiles (scoped to the demo cohort) on MG Core's central S3 infrastructure.

- **Retention period**: Maximum 3 months from data copy, aligned with MG DLP.
- **Deletion trigger**: 3-month expiry and account deactivation events.
- **Current status**: Deletion is manual. Automated deletion pipeline is not yet defined for this environment.
- **Planned approach**: Automated deletion will be evaluated after the demo iteration phase concludes and the project's long-term scope is determined. Current plan is to bulk-delete all demo environment data upon completion of demo iterations, before entering the production readiness phase. If the project proceeds to production, an automated DLP pipeline will be designed at that stage.
