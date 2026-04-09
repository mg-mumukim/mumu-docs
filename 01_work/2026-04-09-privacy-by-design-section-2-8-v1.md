# Privacy by Design - Section 2.8 Draft

Target document: `00_note/2026-04-03-privacy-by-design-aura-demo-wip.md`

---

## 2.8 How long will the data be retained before it is anonymized or deleted?

Response:

All source data is governed by Match Group's Data Lifecycle Policy (DLP). All environments apply a maximum 3-month retention period for Personal Information. The project operates across three environments with different retention mechanisms, summarized below and detailed per environment.

| Environment | Retention | Deletion | Automated? | Verification |
|-------------|-----------|----------|------------|-------------|
| HyperPod (model training) | 3 months / account event | DLP pipeline | Phase 1: manual; Phase 2: automated (under development) | S3 inventory snapshot |
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
