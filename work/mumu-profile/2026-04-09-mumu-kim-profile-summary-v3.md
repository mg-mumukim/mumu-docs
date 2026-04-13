# Mumu Kim - Profile Summary

> Sources:
> - Slack activity analysis (mgai-aura, mgai-chemistry, mgai-leaders, mgai-general, mgai-tailor-mlse, ml-team-gai, ml-photo-review, mgai-prompt-factory, + Glean search for cross-team channels)
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

#### A2. On-device ML Platform (OPaaS)

Platformized Hyperconnect's on-device ML, reducing feature introduction time from 6 months.

- Model conversion scripts (CoreML/TFLite) for iOS/Android
- Tinder sprint: PoC app development, vendor SDK benchmarking (face recognition)
- Hinge sprint: CI pipeline, Android SDK development, model conversion/encryption scripts
- Cross-brand collaboration: Tinder, ASL, Hinge

#### A3. On-premises Training Infrastructure (DGX)

Managed ~7B KRW GPU cluster: hardware repair coordination (25 cases in 2023), user experience fixes (storage, SSH, Slurm, OOM), and ~400M KRW warranty renewal contract.

### B. Hurricane Project - GDPR Data Cleanup (2024 H1)

Coordinated organization-wide cleanup of personal user data in ML training data for GDPR compliance. Tracked and cleaned key training data across cloud and on-premises environments while preserving as much moderation data as possible.

### C. LLM Serving Infrastructure (2024 H1)

Designed and introduced a system to run LLMs on existing on-premises GPU cluster for the VH chatbot service, avoiding ~30M KRW/month cloud costs. Estimated savings of hundreds of millions of KRW over 4 months pre-launch. Presented at MLOps Now (external) and internal AI seminars.

### D. Hinge Acceleration (2025 H1)

#### D1. Top Photo v3 & Photo Finder

Led ML-focused projects with MLE members (Seren, Eden, Link) and MLSE member (Joel). Shipped both on schedule. Handled on-device model preprocessing/conversion/quantization and CyberLink updates directly.

- Established delegation foundation: transferred PM responsibilities to Juniper, defined role divisions, documented PM practices (kanban)
- Subsequent projects (MPaaS fairness evaluation, Hinge Photo Coaching) run by Juniper with Mumu sharing ML management

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
| Kangsoo Lee | Engineer (Tinder Seoul) | CRS iOS engineering, cross-team sync |
| Yuza Lee | PM (Tinder Seoul) | Chemistry launch coordination, Australia deployment, experiment management |

### Hinge

| Name | Role | Focus Areas |
|------|------|-------------|
| Seren Eom | MLE | Top Photo v3, Photo Finder |
| Eden Choi | MLE | Top Photo v3, Photo Finder |
| Joel Ham | MLSE | Optimization, insight matching deployment, ML System Unit |

### Cross-team (Tinder / MG Core)

| Name | Role | Focus Areas |
|------|------|-------------|
| Jason Park | SRE / MLSE | GitHub Enterprise, mg-core infra coordination, security review |
| Trideep Rath | PM (Tinder Gen AI) | Photo Review product alignment, Gemini team coordination |
| Jaini Shah | PM (Tinder) | Photo Review product brief, design specs |
| Ben Shin | TL (Tinder Gen AI) | GenAI x MGAI weekly sync, AURA workshop coordination |
| Skanda Suresh | MLE (Tinder Gen AI) | SmartPhotos v4, rSRR dataset, CRS ML optimization collaboration |
| Harvey Ko | MLSE (Pairs) | Real-time recommendation backend, DBX MLflow model registry |
| Diego Shin | MLSE | PII redaction server, ML System Unit |
| Enkhee Erdenee | MLE | Mentee (soft skills, career discussions) |
