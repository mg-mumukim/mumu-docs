# Mumu Kim - Profile Summary

> Sources: Slack (Tinder workspace), Notion (hpcnt workspace), employee evaluations (2023-2025), Glean search
>
> Period: 2023 ~ 2026-04

---

## 1. Overview

| Item | Detail |
|------|--------|
| Organization | MG AI (Match Group AI), Tinder / Hyperconnect |
| Role | Senior IC |
| Career Path | ML Platform (2020~2023) -> MG AI Dev (2024~) |
| Manager | Shurain Ha |
| Peers | Owen Lee (MLSE), Juan (MLE), Harriet (PM) |

Manager characterization: "solver/architect hybrid for high-leverage ML problems." Operates at multiple abstraction levels. Excels at enabling work more than coordinating execution.

---

## 2. Work Domains

Each domain is tagged with primary involvement type:
- **Build**: Wrote code, trained models, built tools directly
- **Author**: Wrote documents, specs, or plans as author
- **Lead**: Planned work, delegated tasks, managed execution
- **Review**: Reviewed others' designs, code, or specs
- **Coordinate**: Drove cross-team alignment, unblocked dependencies

---

### A. On-device ML & Model Engineering

**Primary role: Build + Lead**

This is Mumu's deepest hands-on domain. He writes model conversion/training code, builds evaluation tools, and directly debugs on-device issues -- while also planning the ML direction and delegating subtasks to MLEs.

**OPaaS Platform (2023~2025)**
- `Build` Model conversion scripts (CoreML/TFLite) for iOS/Android, CI pipeline, vendor SDK benchmark app
- `Build` Platformized Hyperconnect's on-device ML, cutting feature introduction time from 6 months
- `Coordinate` Cross-brand collaboration: Tinder, ASL, Hinge sprints

**Hinge Top Photo & Photo Finder (2025 H1)**
- `Build` On-device model preprocessing, conversion, quantization; CyberLink SDK updates
- `Build` Photo Finder demo app for team-wide qualitative evaluation (vibe check)
- `Lead` Led ML projects with Seren, Eden, Link (MLE) and Joel (MLSE); shipped on schedule
- `Lead` Transferred PM responsibility to Juniper; established delegation model and kanban practices
- `Author` Top Photo tech blog (rejected by Hinge comms due to brand concerns -- led to anonymization discussion)
- `Coordinate` Throughput investigation with Hinge team; iOS vs Android performance analysis

**Camera Roll Scan v2/v3 (2025 H2 ~ 2026 Q1)**
- `Build` Trained MobileNetV3 Small/V4 backbones on 32 H100 GPUs (CLIP distillation on DataCompDR-12M) -- motivated by MobileCLIP being 61x slower on Android
- `Build` Benchmark app (camera-roll-scan-benchmark), quantized MobileCLIP, qualitative evaluation Mac app (with Brandon)
- `Lead` Proposed and led 3x end-to-end speed optimization project (with Skanda, Claude); parallelized backbone/head training across engineers
- `Lead` Proposed 1024-dim to ~128-dim embedding reduction for latency (delegated to Groot/Joel)
- `Review` Tinder Seoul engineering docs (Kangsoo Lee), CRS v4 pipeline rearrangement
- `Coordinate` Australia deployment, Tinder Recs integration, moderation model reuse investigation (Skanda's model + Hinge face detection)

---

### B. System Design & Architecture

**Primary role: Author + Review**

Mumu proactively writes architectural documents before being asked. He reviews others' specs with attention to diagrams, terminology precision, and alternative approach comparison.

**AURA Backend Contract**
- `Author` Drafted and shared before cross-team kickoff with SOUP team
- `Review` Prompt Factory RFC (terminology, architecture, alternative approaches)

**Photo Review Serving**
- `Advisor` Photo Review Serving Optimizations & Remaining Questions document (Brandon's)
- `Coordinate` Feature Forum leadership presentation preparation for Tinder Core leadership

**CRS v3 Proposal**
- `Author` CRS benchmark result documentation for Tinder (iOS/Android breakdown, optimization roadmap)
- `Lead` Terminology standardization: replaced opaque names ("model 1", "prompt 2") with functional roles

---

### C. Privacy, Compliance & Data Governance

**Primary role: Author + Coordinate**

Mumu drives privacy documentation and data governance processes. He writes the actual documents and coordinates with legal teams.

**AURA Privacy by Design (2026 Q2)**
- `Author` Lead author of PBD document: data processing, DLP, legal compliance
- `Coordinate` Legal review sessions (GenAI sync), DLP policy definition

**Hurricane GDPR Cleanup (2024 H1)**
- `Coordinate` Organization-wide ML training data cleanup for GDPR compliance
- `Build` Identified EEA user edge cases (active users needing content deletion), cross-referenced GDPR_users/closed_users/Users tables
- `Coordinate` DWH team (SK Na, Jerry Lee) for deletion criteria and fallback strategies

---

### D. Infrastructure & Compute

**Primary role: Build (early) -> Coordinate (recent)**

Started as hands-on infra operator; transitioned to coordination role as team scaled.

**DGX On-premises Cluster (2023~2025)**
- `Build` Managed ~7B KRW GPU cluster (20x A100): 25 hardware repair cases, user experience fixes (storage, SSH, Slurm, OOM)
- `Author` ~400M KRW warranty renewal: documented full procurement procedure (BEF, vendor registration, CAF)
- `Build` DGX H100 procurement evaluation: price/performance comparison across alternatives
- `Coordinate` Cluster migration to HyperPod (2025-12): adjusted training plans (2 A100 nodes -> 1 H100 node)

**LLM Serving (2024 H1)**
- `Build` Designed and built LLM serving system on DGX for VH chatbot; saved ~30M KRW/month vs cloud
- `Author` Presented at MLOps Now (external) and internal seminars

---

### E. Cost & Resource Planning

**Primary role: Author**

- `Author` LLM cost estimation: traffic-based (LP-485) and GPU node-based (LP-486)
- `Author` MLSE resource planning: authored and shared to GenAI team

---

### F. Team & Project Management

**Primary role: Lead**

Mumu runs the daily/weekly cadence for AURA squad and coordinates MLSE resource allocation across projects.

**Squad Operations**
- `Lead` AURA daily sync, PM-TL weekly sync (with Juniper), JIRA epic/issue management
- `Lead` MLSE resource planning across Hinge Photo Finder, HingeX Photo Analyzer, MPaaS fairness evaluation, CRS, AURA
- `Coordinate` MG AI onsite (Palo Alto 2026-03-09~13), Vancouver global onsite preparation (2026-05-11~15)

**Delegation Model**
- Juniper Han: PM responsibilities fully transferred; now leads projects independently
- Brandon Choi: Mature delegation on iOS/Android/Server implementation
- Owen Lee: ML System Unit server-side execution (PII redaction, insight matching) with periodic check-ins

---

### G. Organizational Improvement

**Primary role: Drive + Propose**

- Retrospective culture: established project-level retrospectives and retrospective-of-retrospectives
- Engineering principles: proposed to compensate for absent engineering culture
- MLSE structure: proposed system/innovation unit separation, manager/TL separation
- Process: Jira visibility rules, meeting time reduction, document template standardization
- Hiring: JD drafting (MLSE 2025), interview question design (with Lana Na)
- Education: proposed "missing semester" program for MLE engineering skills; ML study groups (RecSys, Ultra-Scale Playbook)

---

## 3. How Mumu Works

### Technical Style

- Prefers solving problems at the representation level: problem formulation, metric design, evaluation framework
- Systematic ML study (causal inference -> probability -> RecSys -> LLM training) to collaborate precisely with MLEs
- Proactively authors documents (backend contracts, privacy docs, cost estimates) before they are requested
- Demands clear diagrams, precise terminology, and explicit alternative approach comparison in reviews

### Communication Style

- "Important detail hunter" (Juniper Han)
- "Uniquely valuable in cross-team settings" (Manager)
- Prefers concrete examples over abstract descriptions
- Requests "less is better / more is better" interpretation upfront for metrics
- Direct follow-up culture: names individuals, sets deadlines, tracks via Jira and Notion

### Documentation Practices

- Notion (hpcnt workspace): meeting notes, project specs, experiment logs, onboarding docs, personal thought journal
- Google Docs: cross-team documents for Tinder stakeholder visibility
- Jira: work tracking and visibility (rules enforcement)
- Slack: proactive SSOT link sharing, action item summaries from meetings

---

## 4. Channels & Visibility

**MG AI Internal**: mgai-leaders, mgai-aura, mgai-chemistry, mgai-general, mgai-tailor-mlse, mgai-prompt-factory, mgai-dev, mgai-smalltalk

**Tinder Cross-team**: ml-team-gai, ml-team, ml-photo-review, photo-review, tinder-litellm, smart-photos-v4, smart-photos-v4-ml, chemistry-leads-team, chemistry-dev, chemistry-camera-roll-tagging, chemistry-mgai-tinder-seoul, tailor-mg-ai, mg-core-infra-ai, mgai_hinge_photo_picker_dev, mgai-tinder-recs, dev-hurricane-tf

**Other**: ai_paper, mg_seoul

---

## 5. People

### MGAI Leadership

| Name | Role | Relationship |
|------|------|-------------|
| Harriet Son | PM Manager | Reports-to; AURA/Chemistry updates, onsite logistics |
| Owen Lee | TL, MLSE | Peer TL; MLSE resource allocation, infra decisions |
| Shurain Ha | TL | Peer TL |

### AURA Squad

| Name | Role | Mumu's interaction |
|------|------|-------------------|
| Biggie Choi | ML Lead | ML direction alignment, coaching/concept strategy |
| Juniper Han | PM | PM-TL sync counterpart; Mumu delegated PM to her |
| Brandon Choi | MLSE | Primary delegation partner for implementation (demo, backend, iOS/Android) |
| Parker Cho | MLSE | Prompt Factory, eval pipeline; Mumu reviews direction |
| Claude Yoon | MLE | Concept extraction, VLM; Mumu mentored onboarding and reviews ML plans |
| Ken Lee | MLE | Concept evaluation; Mumu sets evaluation scope |
| Link Lee | MLE | rSRR analysis, demographic concepts; Mumu mentors ML concepts and soft skills |

### Chemistry / Tailor

| Name | Role | Mumu's interaction |
|------|------|-------------------|
| Jiwon Choi | MLSE | CRS server-side, photo collage; Mumu reviews and coordinates |
| Juan Kwon | MLSE | CRS pipeline; Mumu co-planned resource allocation |
| Groot Ko | MLSE | PII redaction model, embedding conversion; Mumu delegated training tasks |
| Kangsoo Lee | Eng (Tinder Seoul) | CRS iOS; Mumu reviewed engineering docs and proposed fixes |
| Yuza Lee | PM (Tinder Seoul) | Chemistry launch; Mumu coordinated Australia deployment |

### Hinge

| Name | Role | Mumu's interaction |
|------|------|-------------------|
| Seren Eom | MLE | Top Photo v3; Mumu led project |
| Eden Choi | MLE | Photo Finder, MobileCLIP training; Mumu co-worked on model training |
| Joel Ham | MLSE | Optimization, Databricks; Mumu delegated insight matching and mentored |

### Cross-team (Tinder / MG Core)

| Name | Role | Mumu's interaction |
|------|------|-------------------|
| Jason Park | SRE / MLSE | GitHub Enterprise, infra coordination; Mumu coordinates |
| Trideep Rath | PM (Tinder GenAI) | Photo Review alignment; Mumu's primary Tinder PM counterpart |
| Jaini Shah | PM (Tinder) | Photo Review specs; Mumu discusses requirements directly |
| Ben Shin | TL (Tinder GenAI) | GenAI x MGAI sync; Mumu's weekly cross-team counterpart |
| Skanda Suresh | MLE (Tinder GenAI) | SmartPhotos, CRS collaboration; Mumu co-planned ML optimization |
| Harvey Ko | MLSE (Pairs) | Recs backend, Databricks; Mumu coordinates and reviews runbook |
| Diego Shin | MLSE | PII redaction server; Mumu delegates via Owen |
| Enkhee Erdenee | MLE (Tinder Recs) | Mumu mentors (soft skills, career) |
| Ke Liu | MLE (ML Platform) | HyperPod/DGX cluster coordination |
