# Mumu Kim - Profile Summary

> Sources:
> - Slack activity analysis (mgai-aura, mgai-chemistry, mgai-leaders, mgai-general, mgai-tailor-mlse, ml-team-gai, ml-photo-review, mgai-prompt-factory, mgai-dev, dev-hurricane-tf, mgai_hinge_photo_picker_dev, ml-team, + Glean search)
> - Employee evaluation summaries (2023, 2024, 2025)
>
> Period: 2023 ~ 2026-04

---

## 1. Overview

| Item | Detail |
|------|--------|
| Organization | MG AI (Match Group AI), Tinder / Hyperconnect |
| Department | Data Science / ML / AI |
| Role | Tech Lead (TL), Senior Engineer (IC) |
| Career Path | ML Platform / Infra (2023) -> MG AI Dev (2024) -> MG AI TL (2025~) |
| Responsibilities | PM-TL sync facilitation, MLSE weekly management, squad daily operation, tech spec review, cross-team coordination |

---

## 2. Related Projects

### A. ML Platform & Infrastructure (2023)

#### A1. Data Pipeline (MPaaS)

Standardized and automated media report data processing for ML training (100M+ samples).

- System design, pipeline infrastructure setup, data processing implementation
- Covered multiple media domains: audio, image, text, video reports
- Handled major service migration (AzarAIMO -> MPaaS) without issues
- Used by AIP ML team for model updates (HumanVision, Antman, SafeSpeechContext) and Hinge moderation datasets
- MPaaS fairness evaluation continued into 2025 as ongoing workstream (codebase: mpaas-fairness on DGX cluster)

#### A2. On-device ML Platform (OPaaS)

Platformized Hyperconnect's on-device ML, reducing feature introduction time from 6 months.

- Model conversion scripts (CoreML/TFLite) for iOS/Android
- Tinder sprint: PoC app development, vendor SDK benchmarking (face recognition)
- Hinge sprint: CI pipeline, Android SDK development, model conversion/encryption scripts
- Cross-brand collaboration: Tinder, ASL, Hinge
- OPaaS License management and renewal; demo application separation development (2025)
- Post-Hinge Photo Finder: continued as SDK platform for CRS v2/v3 pipeline (tools repo: opaas-model-tools)

#### A3. On-premises Training Infrastructure (DGX)

Managed ~7B KRW on-premises GPU cluster (20x A100).

- Hardware repair coordination (25 cases in 2023), user experience fixes (storage, SSH, Slurm, OOM)
- ~400M KRW warranty renewal contract: identified procedures (BEF, vendor registration, CAF, purchase orders) and documented
- DGX H100 procurement evaluation: researched options, compared price/performance across alternatives for 2024 compute resources
- DGX cluster opened to Tinder ML team (2025-08, coordinated by Ke Liu): Mumu immediately notified MGAI team for access
- DGX terminated and migrated to HyperPod cloud cluster (2025-12-15): Mumu adjusted training plans from 2 A100 nodes to 1 H100 node during transition

### B. Hurricane Project - GDPR Data Cleanup (2024 H1)

Coordinated organization-wide cleanup of personal user data in ML training data for GDPR compliance.

- Tracked and cleaned key training data across cloud and on-premises environments while preserving as much moderation data as possible
- Coordinated via dev-hurricane-tf channel with DWH team (SK Na, Jerry Lee, Albert Jung, Jake Ryu)
- Investigated EEA user identification: queried about registration_country_code NULL cases (~220K users), discussed fallback strategies with DWH
- Identified edge case: active (non-withdrawn) users in EEA still requiring content deletion (profile, nickname) -- separate from closed_users table
- Cross-referenced GDPR_users table, closed_users table, and Users table to ensure comprehensive coverage

### C. LLM Serving Infrastructure (2024 H1)

Designed and introduced a system to run LLMs on existing on-premises GPU cluster for the VH chatbot service.

- Avoided ~30M KRW/month cloud costs by leveraging existing DGX cluster instead of cloud LLM providers
- Estimated savings of hundreds of millions of KRW over 4 months pre-launch
- Presented at MLOps Now (external speaker) and internal AI organization seminars
- Note: Pre-Tinder-Slack period; limited Slack evidence available for this project

### D. Hinge Acceleration (2025 H1)

#### D1. Top Photo v3 & Photo Finder

Led ML-focused projects with MLE members (Seren, Eden, Link) and MLSE member (Joel). Shipped both on schedule. Handled on-device model preprocessing/conversion/quantization and CyberLink updates directly.

- Established delegation foundation: transferred PM responsibilities to Juniper, defined role divisions, documented PM practices (kanban)
- Subsequent projects (MPaaS fairness evaluation, Hinge Photo Coaching) run by Juniper with Mumu sharing ML management

#### D2. Slack-sourced details

- **Photo Finder demo app** (2025-04): Developed demo app and deployed team-wide for vibe check evaluation. Entire MGAI team tested with their own camera rolls. Analyzed scoring pipeline failures -- identified gender-mix filtering, solo photo scoring issues, and mirror-shot biases. Discussion led to Top Photo v3 problem definition: whether to optimize SR lift with table data only or incorporate qualitative "human perception" aspects.
- **Photo Finder throughput** (2025-05): Investigated performance with Hinge team (Kevin Sun). ~1400 photos scanned in 25s on Pixel 7 Pro. Raised concern about iOS vs Android performance discrepancy -- "Android didn't have any problem. Can we check again if Android performance was really okay?"
- **Top Photo tech blog** (2025-07): Written and submitted to Hinge comms for review. Rejected due to brand concerns: Hinge emphasized meaningful connections over appearance optimization; missing fairness/guardrails/bias discussion; insufficient emphasis on MG collaboration. Led to brand anonymization consideration.
- **Jira board consolidation** (2025-04): Proposed merging Hinge and Central (OPaaS) boards. Team agreed to use one board with Epic-level separation, reducing overhead.
- **MLSE resource planning** (2025-06): After Photo Finder v2, planned resource distribution across Hinge Photo Finder, HingeX Photo Analyzer, and MPaaS fairness evaluation. Advocated for MLSE members to take full initiative ownership.
- **Hinge model code migration** (2025-07): Coordinated migration of scouter-hinge, hinge-photo-ordering, hinge-photo-finder repos from GHES to GHEC during GitHub org migration.
- **Cross-team channel**: mgai_hinge_photo_picker_dev

### E. Tinder Tailor / Chemistry - Camera Roll Scan (2024 H2 ~ 2025)

On-device ML feature that scans users' camera rolls to recommend photos for their dating profiles.

- CRS v2/v3 problem definition, metrics design, optimization planning
- On-device latency optimization, Android device tier coverage analysis
- CRS v4 pipeline rearrangement tech spec review
- Tinder Recs team collaboration: Camera Roll integration sync
- Tailor release coordination (Australia deployment)
- PM-TL sync content preparation and cross-squad communication

#### Evaluation-sourced details (2025 H2):

- Delivered prototyped ML pipeline to Tinder production; identified and resolved gaps in pipeline evaluation
- Improved de-duplication logic, on-device model/preprocessing logic, clustering parameters, implementation bugs
- Cross-domain execution (Client/Backend/ML): qualitative evaluation Mac app (with Brandon), moderation requirements research, quantized MobileCLIP, Tinder Seoul engineering doc review, server-side PII redaction decision
- Proposed and led 3x end-to-end speed optimization project (with Skanda and Claude) including backbone training, head training parallelization
- Latency reduction: proposed reducing 1024-dim to ~128-dim insight matching embeddings (delegated to Groot/Joel)

#### Slack-sourced details:

- **MobileCLIP Android problem** (2025-11): Apple MobileCLIP was 61x slower on Android than on iOS, making it unusable. This motivated training a custom MobileNetV3-based backbone as a replacement.
- **Model training** (2025-11~12): Directly trained MobileNetV3 Small/V4 backbones using CLIP/SigLIP2 distillation on DataCompDR-12M dataset. Used 32 H100 GPUs on HyperPod. Debugged training issues with Eden (data loading errors, GPU memory). Trained two variants: V4 (ambitious target) and V3 (backup).
- **Moderation model reuse** (2025-09~10): Investigated reusing Tinder's on-device moderation model (Skanda's work) and Hinge face detection model (Eden's work) for CRS PII redaction. Proposed building PII redaction model based on Hinge's face detection approach extended to text detection.
- **Legal/data coordination** (2025-11): Drove data access clarification for training: rSRR dataset and moderation dataset from Tinder. Pushed for explicit data access assumptions (storage location, retention, access control) beyond just requisition in legal docs.
- **CRS benchmark app**: Developed camera-roll-scan-benchmark repository for iteration evaluation. Used by Parker for prompt iteration with fixed datasets.
- **Benchmark doc for Tinder CTO** (2025-11): Coordinated benchmark results documentation (Google Docs) for Tinder CTO visibility, with iOS/Android breakdown by photo count, pipeline stages, and optimization roadmap.
- **Terminology standardization** (2025-11): Pushed for replacing opaque names (e.g., "model 1", "prompt 2") with functional role names (cluster description generation, insight consolidation).

### F. AURA - AI Profile Enhancement (2025 Q4 ~ present, primary focus)

AI agent system for profile enhancement. Ultimate goal: an agentic dating coach that plans, intervenes, and learns from experience.

#### F1. System Design & Backend

- Backend contract draft: proactively authored and shared before cross-team kickoff (with SOUP team)
- Multi-brand system design consideration: Pairs Japan Q3 adoption planning
- Prompt Factory: RFC review (terminology, architecture, alternative approach analysis)

#### F2. Privacy & Compliance

- Privacy by Design (PBD) document: lead author (LP-360), covering data processing, DLP, legal compliance
  - Section 2.8: data retention/deletion policy across HyperPod, local machines, MG Core S3
  - DLP: manual bulk deletion (3-month retention) + automated infra-level deletion (in progress)
  - Legal review coordination (GenAI sync)
- Data management: HyperPod cluster Tinder S3 access, DLP retention policy

#### F3. ML / Concept / Coaching

- Photo Review: ML-side planning, serving optimization engineering doc coordination
- Knowledge base extraction: scope extends beyond AURA demo into Photo Review input
- rSRR-based lifestyle concept: demographic features (bio, interest, descriptor) to new coaching concept addition (post-4/10 scope)
- Coaching & concept extraction alignment: LLM-based coaching vs ML model-based concept gap identified
- Terminology standardization: "Profile Photo" vs "Profile" disambiguation, glossary initiative participation

#### F4. Cost & Resource Planning

- LLM cost estimation documents: traffic-based (LP-485) and GPU node-based (LP-486)
- LLM+GPU cost calculator dashboard (LP-493)
- MLSE resource planning: authored and shared to GenAI team

#### F5. Cross-team & Operations

- Gemini team integration (via Trideep Rath)
- Squad operation: daily sync, weekly PM-TL sync, JIRA epic/issue management
- MG AI onsite trip (Palo Alto, 2026-03-09~13)

### G. Infrastructure & Platform (ongoing)

- HyperPod cluster: data access architecture (IAM roles, FSx/S3 sync)
- Databricks workspace: cursor-to-workspace sync exploration
- LiteLLM: team-wide adoption facilitation, key migration
- GitHub organization: hyperconnect to mg-ai migration planning

---

## 3. Core Competencies

### Technical

| Area | Detail |
|------|--------|
| ML System Engineering | Training infrastructure (HyperPod, DGX), model serving, on-device ML pipeline (CoreML/TFLite), model conversion/quantization |
| System Design | Multi-brand architecture, backend API contract design, serving optimization, data pipeline design |
| Cloud / Infra | AWS (S3, IAM, HyperPod, FSx), Databricks, Kubernetes, GitHub Enterprise, on-prem GPU clusters |
| LLM Engineering | Cost optimization (traffic/GPU), LiteLLM proxy, prompt engineering review, LLM serving on on-prem GPU |
| Privacy & Compliance | Privacy by Design authoring, Data Lifecycle Policy, legal coordination, GDPR data cleanup |

### Problem Structuring & Technical Leadership

Manager characterization: "solver/architect hybrid for high-leverage ML problems."

- Operates at multiple abstraction levels -- from implementation details to strategic framing
- Intuition for abstract structures, ability to concretize problems, asking questions at the right depth
- Problem structuring creates leverage: not just unblocking individual projects but shaping how the team thinks about hard problems
- Excels more at enabling work than coordinating execution

### Mentoring & Delegation

| Mentee | Area |
|--------|------|
| Juniper Han | Project management delegation; Juniper now leads projects independently |
| Joel Ham | Optimization techniques |
| Claude Yoon | Onboarding |
| Link Lee | ML concept mentoring, soft skills |
| Enkhee Erdenee | Soft skills, career discussions |
| Brandon Choi | Built mature delegation model on iOS/Android and ML System Unit |

### Organizational Improvement

- Retrospective culture: project-level retrospectives, retrospective-of-retrospectives
- Engineering principles: proposed to compensate for absent engineering culture
- Jira visibility: established and maintained rules for work tracking
- MLSE structure: proposed system/innovation unit separation, manager/TL separation
- Meeting optimization: reduced unnecessary recurring meeting times, limited participants
- Document templates: catalyzed creation by pointing out inconsistencies (Juan, Harriet)
- Proposed "missing semester" program for MLE engineering skills development
- Hiring: JD drafting (MLSE 2025), interview question design, recruiter pipeline coordination (with Lana Na)

### ML Self-Study Trajectory

Systematic study path viewing ML as an independent discipline:

- Fundamentals: causal inference, information theory, probability
- Textbook: Probabilistic Machine Learning
- Practical: Machine Learning Engineering in Action
- Study groups: recommendation systems (Netflix/YouTube/Meta)
- Current: The Ultra-Scale Playbook (LLM training on GPU clusters), Reinforcement Learning (Algorithms for Decision Making)

### Leadership (Operations)

| Area | Detail |
|------|--------|
| Technical Review | Thorough RFC/tech spec review. Demands: clear diagrams, precise terminology, and explicit alternative approach comparison |
| Cross-team Coordination | Legal team, Tinder Gen AI, SOUP backend, Pairs JP, Hinge |
| Meeting Facilitation | PM-TL sync, squad daily, squad discussion (agenda-driven, knowledge sharing, proposal review) |
| Resource Management | MLSE resource planning, GPU/LLM cost budgeting |
| Documentation | Proactive authoring (backend contracts, privacy docs, cost estimates before asked) |

### Communication Style

- "Important detail hunter" -- Juniper Han
- Manager: "uniquely valuable in cross-team settings"
- Prefers concrete examples over abstract descriptions in documentation
- Requests "less is better / more is better" interpretation upfront for metrics
- Pushes for alternative approach comparison in design reviews
- Direct follow-up culture for review requests

---

## 4. Channels & Visibility

### MG AI Internal

| Channel | Role |
|---------|------|
| mgai-leaders | Leadership sync, strategic decisions, resource allocation |
| mgai-aura | Primary project channel, daily operations |
| mgai-chemistry | Tailor/CRS technical discussions |
| mgai-general | Team-wide announcements, tooling, org matters |
| mgai-smalltalk | Team social channel |
| mgai-tailor-mlse | MLSE weekly sync, infra coordination |
| mgai-prompt-factory | Prompt Factory RFC and development |
| mgai-dev | Engineering development discussions |

### Tinder Cross-team

| Channel | Role |
|---------|------|
| ml-team-gai | Tinder Gen AI cross-team collaboration (MGAI x GenAI weekly sync) |
| ml-team | Tinder ML team main channel (DGX cluster access, infra coordination) |
| ml-photo-review | Photo Review ML discussions (created by Trideep Rath for MLE/MLSE technical discussion) |
| photo-review | Photo Review product-level cross-team channel (Jaini Shah, Trideep Rath, Henry Au) |
| tinder-litellm | LiteLLM enterprise offering coordination (legal, security, Tinder engineering) |
| smart-photos-v4 | SmartPhotos v4 LLM service operations (backfill, cache, GPU scaling) |
| smart-photos-v4-ml | SmartPhotos v4 ML-focused discussions (capacity, model serving) |
| chemistry-leads-team | Chemistry feature leads channel (TJ Purtell, Yuza Lee, Jenn Han) |
| chemistry-dev | Chemistry feature development (Tinder Seoul, launch coordination) |
| chemistry-camera-roll-tagging | CRS cross-team sync (MG AI x Tinder Seoul, Kangsoo Lee) |
| chemistry-mgai-tinder-seoul | Chemistry cross-team collaboration (MGAI x Tinder Seoul) |
| tailor-mg-ai | Tailor feature cross-team channel (Lucas Newcomer, Meha Garg, Thomas Yoon) |
| mg-core-infra-ai | MG Core Infra x MG AI coordination (LiteLLM, security review, Jason Park) |
| mgai_hinge_photo_picker_dev | Hinge Photo Finder cross-team development (Kevin Sun, Doug Galante) |
| mgai-tinder-recs | Tinder Recs related discussions |
| dev-hurricane-tf | Hurricane/GDPR data cleanup coordination |

### Other

| Channel | Role |
|---------|------|
| ai_paper | Paper sharing and discussion |
| mg_seoul | MG Seoul office-wide channel |

---

## 5. People

### MGAI Leadership

| Name | Role | Relationship to Mumu | Focus Areas |
|------|------|---------------------|-------------|
| Harriet Son | PM Manager | Reports-to chain; shares AURA/Chemistry updates | Team operations, onsite logistics, cross-team PM coordination, Chemistry product oversight |
| Owen Lee | TL, MLSE | Peer TL; MLSE resource allocation counterpart | MLSE across multiple squads, infra decisions (DBX, security review), Recs/Pairs/Chemistry |
| Shurain Ha | TL | Peer TL | -- |

### AURA Core

| Name | Role | Focus Areas |
|------|------|-------------|
| Biggie Choi | ML Lead | ML technical direction, eval framework, coaching/concept strategy, team-level decision-making |
| Juniper Han | PM | Product spec, evaluation criteria, coaching direction, Tinder leadership communication (Trideep, Jaini) |
| Brandon Choi | MLSE | AURA demo/web, backend implementation, coaching logic, repetitive photo detection |
| Parker Cho | MLSE | Prompt Factory development, DSPy judge optimization, evaluation pipeline, CRS rescan |
| Claude Yoon | MLE | Concept extraction, VLM, ML model reports, ticket prioritization |
| Ken Lee | MLE | Concept evaluation (T2 sanity check), labeling, model analysis |
| Link Lee | MLE | rSRR analysis, demographic concept research, knowledge base analysis |

### Chemistry / Tailor

| Name | Role | Focus Areas |
|------|------|-------------|
| Jiwon Choi | MLSE | On-device ML (CRS), server-side ML, photo collage, observability logging |
| Juan Kwon | MLSE | CRS pipeline, document templates |
| Groot Ko | MLSE | Predibase migration, PII redaction model training, embedding model conversion |
| Kangsoo Lee | Engineer (Tinder Seoul) | CRS iOS engineering, cross-team sync |
| Yuza Lee | PM (Tinder Seoul) | Chemistry launch coordination, Australia deployment, experiment management |

### Hinge

| Name | Role | Focus Areas |
|------|------|-------------|
| Seren Eom | MLE | Top Photo v3, Photo Finder |
| Eden Choi | MLE | Top Photo v3, Photo Finder, MobileCLIP training, face detection model |
| Joel Ham | MLSE | Optimization, insight matching deployment, ML System Unit, Databricks expertise |

### Cross-team (Tinder / MG Core)

| Name | Role | Focus Areas |
|------|------|-------------|
| Jason Park | SRE / MLSE | GitHub Enterprise, mg-core infra coordination, security review |
| Trideep Rath | PM (Tinder Gen AI) | Photo Review product alignment, Gemini team coordination |
| Jaini Shah | PM (Tinder) | Photo Review product brief, design specs |
| Ben Shin | TL (Tinder Gen AI) | GenAI x MGAI weekly sync, AURA workshop coordination |
| Skanda Suresh | MLE (Tinder Gen AI) | SmartPhotos v4, rSRR dataset, CRS ML optimization collaboration |
| Harvey Ko | MLSE (Pairs) | Real-time recommendation backend, DBX MLflow model registry, runbook template |
| Diego Shin | MLSE | PII redaction server, ML System Unit |
| Enkhee Erdenee | MLE | Tinder Recs, mentee (soft skills, career discussions) |
| Ke Liu | MLE (ML Platform) | HyperPod/DGX cluster access coordination, training platform guideline |
