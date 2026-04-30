| Key           | Value                                                                                           |
| ------------- | ----------------------------------------------------------------------------------------------- |
| Source        | `https://docs.google.com/document/d/1hBVeDfT2gm0marF5f0E1ukic1sJw7tRMOiV6Z5Fh6Ow/edit?usp=drivesdk` |
| Downloaded at | `2026-04-23 11:00:00`                                                                           |

---

# Overview

# MG AI HyperPod DLP \- Overview {#mg-ai-hyperpod-dlp---overview}

Author: [Owen Lee](mailto:owen.lee@match.com) [Jason Park](mailto:jason.park@match.com)  
Last Updated: Apr 13, 2026

# Overview

This document defines the DLP (Data Lifecycle Policy) standard for safely and efficiently securing data required for ML model training on MG AI team's infra (especially for Hyperpod \- AWS-based GPU cloud cluster). 

Note that, we don't implement DLP rules from scratch. We fetch closed user accounts/UGCs from each brand, and delete data (table rows and files) according to the closed account/UGC table.

| Data Type | DLP Method | Description |
| :---- | :---- | :---- |
| Tabular data created by MG AI (e.g., user feature data). | A) Row-level deletion | Uses Databricks/Delta Lake's row-level deletion to remove specific records belonging to closed accounts based on standardized column names. A VACUUM command is then executed to physically purge the deleted data files from storage. |
|  | B) Pseudonymize user identifiers | Replaces real user IDs with randomized virtual IDs and manages their connection through a secure mapping table. When DLP is triggered, only the mapping entry is deleted, rendering the data non-traceable and allowing for its permanent retention. |
| Non-tabular data created by MG AI (e.g., cropped user profile image, audio). | C) Delete files by path convention. | Enforces strict folder and filename conventions that embed user identifiers to ensure unstructured files remain traceable. This allows the system to automatically identify and delete all associated files and directories when a user departure occurs. |
| Any | D) Delete by data retention. | Set all data to expire after a certain period of time. (e.g., 90 days)  |

Diagram:  
![image1](../images/1493e36ad77d.png)

# Background Knowledge

### What is HyperPod?

Managed by the Hyperconnect team, HyperPod is an **AWS-based GPU cloud cluster** featuring 80 NVIDIA DGX H100s. It is designed for heavy ML workloads like image/text modeling and LLMs. Unlike ephemeral on-demand instances (e.g., Databricks notebooks), HyperPod provides persistent access, streamlining the development workflow for ML engineers. Currently, Hyperconnect AI and MG AI teams are co-using HyperPod.

### DLP Policy

When a user account is closed (Account Closed) or a direct deletion request is made (UGC Deleted), the associated data enters a deletion grace period. After the grace period expires, DLP is triggered and the relevant files and data are deleted.

| User Type | Data Type | Retention Period |
| :---- | :---- | :---- |
| **Non-banned users** | UGC, lat/long | 90 days |
|  | Analytics data, Service usage log, Most event data, User attributes | 1 year |
|  | Customer care records, App download related data, Consent opt-ins & outs | 5 years |
|  | Purchase / refund history | 10 years |
| **Banned users** | UGC, lat/long | 2 years |
|  | Others | (same with Non-banned users) |

For full details, refer to the internal DLP policy page. ([doc1](https://drive.google.com/file/d/1ykMM_6DvgyWBSqGmZvMC52fH0HiQugVd/view), [doc2](https://drive.google.com/file/d/1lo5PFSb3W1xqAVMfdTsFS-yx8KTp9Dht/view))

### Data Type

* **Data directly managed by brands (e.g., Tinder)**

  * Raw source data mirrored from brand storage (S3). It includes Profile UGC (Profile images, Bios, Prompt answers, Job/School info), Interaction logs (Chat messages, Match events, Reports), and Demographic info (Name, Gender, Age, etc).

  * Note that in Hyperpod, we don't modify the files to the brand storage (e.g., Tinder S3). Thus, we leave the primary data lifecycle management to the respective brand's own compliance systems.

* **Tabular data created by MG AI (e.g., user feature data).**

  * Structured derived data created by joining layers (User, Pair-level tables). It includes pre-processed feature sets for recommendation models, GPT-labeled chat datasets, etc.

  * This data is required because raw data is hard to use directly in ML perspective. By storing these "training-ready" tables, we avoid repetitive and expensive join operations in Databricks that can take several hours per cycle, significantly increasing engineering velocity.

  * Feature examples

    * Recommendation: [https://docs.google.com/spreadsheets/d/1NwjeEnwiYySPkshYcyOPW0zBLzvpxtlNH3X\_ToVLqdk/edit?gid=85206426\#gid=85206426](https://docs.google.com/spreadsheets/d/1NwjeEnwiYySPkshYcyOPW0zBLzvpxtlNH3X_ToVLqdk/edit?gid=85206426#gid=85206426) 

    * Trust\&Safety (Risk-based retrieval): [https://docs.google.com/document/d/1T\_8saf\_q5TTCQGgyfeWcH1W3YuIeB3ZJ6RCFqD9yRrI/edit?tab=t.i13o5awbi8nx\#heading=h.3jhienp7nbik](https://docs.google.com/document/d/1T_8saf_q5TTCQGgyfeWcH1W3YuIeB3ZJ6RCFqD9yRrI/edit?tab=t.i13o5awbi8nx#heading=h.3jhienp7nbik) 

* **Non-tabular data created by MG AI (e.g., cropped user profile image, audio).**

  * Unstructured media assets and their derivatives. This includes profile images downloaded from CDNs, cropped/resized photos, and audio/video (future use cases).

  * Downloading hundreds of GBs from external CDNs and running pre-processing (cropping, rotating) every time is extremely resource-intensive. Storing these assets locally prevents redundant network and computation costs.

# DLP Method Details

## Method A) Row-level deletion

*This method is one of the ways to process **tabular data** created by MG AI.*

This method leverages Databricks/Delta Lake’s **row-level** deletion to purge specific records of users who have closed their accounts. By using standardized column names, automated cleanup jobs identify relevant rows and execute the VACUUM command to physically remove deleted data from the storage layer, ensuring no residual data remains.

![image2](../images/eb6df44034f5.png)

Standard column names for deletion targets:

* brand  
* user\_id  
* user\_number  
* target\_user\_id (Counterpart unique ID)  
* target\_user\_number (Counterpart user number)  
* photo\_id (optional) \- handle cases when the user is still active, but the profile image is deleted and closed.

\* If any column of user identifiers (user\_id, user\_number, target\_user\_id, target\_user\_number) contains a closed account row's value, then the record would be deleted.

## Method B) Pseudonymize user identifiers

*This method is one of the ways to process **tabular data** created by MG AI.*

Unlike physical deletion (Method A), which removes entire records, pseudonymization replaces real identifiers with randomized virtual IDs. The mapping between them is stored in a secure Mapping Table. When a user account is closed, only the mapping entry is deleted — the remaining data becomes anonymized and non-traceable, while still available for AI research. 

Once the mapping is permanently destroyed, the data is no longer subject to an individual's deletion request, enabling consistent evaluation sets and long-term model performance tracking. Engineers should pseudonymize all identifiers wherever possible; if pseudonymization is not feasible, the data must follow the physical deletion path (Method A).

![image3](../images/1e6ff78997af.png)

## Method C) Delete files by path convention

*This method is one of the ways to process **non-tabular** data created by MG AI.*

For unstructured data, a strict naming convention (e.g., paths containing uid={id}) is enforced. This allows the system to automatically scan and delete all files and directories associated with a specific user across S3 and local Hyperpod storage. Files that do not follow this convention are flagged and automatically purged to prevent "dark data" accumulation.

Example file path convention:

```py
# Single-user data:
/data/mgai/{project}/uid={uid}__imageid={imageid}.png
/data/mgai/{project}/user_number={user_number}__imageid={imageid}.png

# Multi-user data (e.g., swiper & swipee peered data)
/data/mgai/{project}/uid={uid}__uid={uid_2}__imageid={imageid}.png

# Single user as a directory with multiple images inside:
/data/mgai/{project}/uid={uid}/image1__imageid={imageid}.png
/data/mgai/{project}/uid={uid}/image2__imageid={imageid}.png
/data/mgai/{project}/uid={uid}/image3__imageid={imageid}.png
```

## Method D) Delete by data retention

*This method is one of the ways to process **tabular or** **non-tabular** data created by MG AI.*

If long data retention is not critical for a project, DLP can be applied by setting a retention period (e.g., 90 days or less) for the entire dataset.

* **Tabular data:** For structured table data, partitioning keys (e.g., `datetime`) can be used to efficiently drop old partitions.

* **Non-tabular data (S3):** For data stored in S3, lifecycle rules can be configured on specific paths to automatically delete all objects after a certain period (e.g., 90 days) from their creation date.

# How to Choose DLP Methods

ML Engineers can choose the DLP method that best fits their current project use case.

## Tabular Data

When creating a Databricks UC table, specify the `dlp_method` tag to determine how DLP is applied.

| Method | 'dlp\_method' tag value | Example use cases |
| :---- | :---- | :---- |
| A: Row-level deletion | `row_delete` | Most feature tables |
| B: Pseudonymize user identifiers | `pseudonymized` | Model evaluation datasets, statistical analysis data |
| D: Delete by data retention | `ttl` | Pairs metadata, temporary cache tables |
| Not subject to DLP | `none` | Aggregated tables with no PI, model artifact metadata |

When using `pseudonymized`, the ML engineer must implement the `user_id` → `safe_id` mapping logic in the data processing code. A dedicated SDK will be provided to simplify this.

## Non-tabular Data (S3)

DLP is applied to S3-stored files via one of the following two methods.

| Method | When Applied | Notes |
| :---- | :---- | :---- |
| C: Delete files by path convention | Default. Automatically applied to most S3 files | File paths must follow the naming convention |
| D: Delete by data retention | When specific paths and retention periods are explicitly configured via IaC | Paths with retention policies are exempt from Method C enforcement |
| Not subject to DLP | When specific paths that are not subject to DLP | Paths with retention policies are exempt from Method C enforcement |

## Exceptions

If the data does not contain PI (Personal Information), DLP can be disabled.

* **Tabular data:** Set `dlp_method=disabled` tag when creating the table. A monitoring dashboard is provided to track which tables have DLP disabled, allowing administrators to review them periodically.  
* **Non-tabular data:** Specific file paths can be configured via IaC to be excluded from DLP enforcement (e.g., `**/model-artifacts/**`).

# FAQ

**Q. How does a Databricks-based DLP system enable DLP on Hyperpod?**  
HyperPod utilizes AWS S3 as a storage engine and uses 'FSx for Lustre' as a cache layer to enhance performance. Since Databricks also uses AWS S3 as a storage engine, both systems share the same underlying data source.

When Databricks identifies "closed data" and removes it from S3 while sending an expiry signal to the cache layer (FSx for Lustre), HyperPod will also lose access to that data.

**Q. What happens if an ML engineer doesn't follow the above rules?**  
Violations are automatically detected, triggering a Slack DM warning. Non-compliant files (e.g., file paths that do not contain `uid` or `imageid`) are deleted after a certain time (e.g., 72 hours).

Example Slack workflow:  
![image4](../images/61ac38dd70cc.png)

# Infra Details

# MG AI HyperPod DLP \- Infra Details

Author: [Owen Lee](mailto:owen.lee@match.com) [Jason Park](mailto:jason.park@match.com)  
Last Updated: Apr 13, 2026

Before reading this document, please refer to the [Overview](#mg-ai-hyperpod-dlp---overview) tab in advance.

# Caveat

This document only covers infrastructure configuration from the perspective of the DLP system. ML engineers accessing other brand data stores for model training is out of scope for this document.

*Note:* ML engineers can fetch data using their own credentials via Databricks API, S3 API, GCP BigQuery, etc., for each Brand. This data is also stored in the MG AI data storage (DBX, S3), which is called derived data. Ensuring DLP for derived data for Hyperpod is the purpose of the Hyperpod DLP system.

# Infrastructure Summary

| AWS Account | Service | Owner | Note |
| :---- | :---- | :---- | :---- |
| MG Core / MG AI | Databricks (Unity Catalog) | MG AI | DLP pipeline, system tables |
| MG Core / MG AI | S3 (`mg-ai-uw1-hyperpod-datasets`) | MG AI | MLE derived data for model training (both tabular \+ non-tabular) |
| MG Core / MG AI | S3 (`mg-ai-uw1-s3-system-inventory`) | MG AI | DLP system data |
| Hyperconnect | SageMaker HyperPod, FSx for Lustre | Hyperconnect | GPU cluster, SSD cache. FSx is linked to MG AI S3 via DRA |

**DLP pipeline runs on:** Databricks Workflows (Jobs) / Delta Live Tables (DLT)  
**IaC:** Databricks Asset Bundles (DABs), Terraform (`mg-databricks`, `mgcore-terragrunt`)  
**Brands in scope:** Tinder, Pairs

# End-to-End Data Flow

The following diagram illustrates how brand data flows into MG AI infrastructure and where the DLP system operates.

![image5](../images/0b98da249756.png)

*MG AI Databricks is used for DLP orchestration (deletion pipeline, verification, bad path monitoring). MLE data access to Tinder raw data (S3, Delta tables) continues through existing credentials and channels.*

# Brand Infra Integration

**Why Brand data integration:** DLP needs to pull **closed objects** from Brand-operated systems \-- specifically **users** and **photos** (and related media in the future).

Currently, data for MGAI Hyperpod includes *Tinder* and *Pairs*. For Tinder, Delta Sharing is used **only** for the 4 closed event tables listed below (Micro-Batch streaming with `Trigger.AvailableNow`, running at 1-hour intervals). General Tinder data access (S3 raw data, Delta tables for training/EDA) is not integrated via External Volume or Delta Sharing \-- MLEs continue to access Tinder data through their existing credentials and channels. For Pairs, we are temporarily resolving DLP by applying a 90-day retention to all data, but we plan to allow fetching the closed objects table via BigQuery connection in the future.

Below is the source table for fetching closed objects from Tinder.

| Source table | Why we need it |
| :---- | :---- |
| `published_prod.tinder_events_delta.account_backend_hard_delete` | Single source for receiving closed users (hard delete) |
| `published_prod.tinder_events_delta.profile_delete_photo` | Confirmed profile media removal \-- photo/video actually removed from the profile |
| `published_prod.tinder_events_delta.trust_account_status` | Account banned by the rules engine |
| `published_prod.tinder_events_delta.trust_case_review` | Account closed via trust case review |

# MG AI Infra Configuration

**HyperPod derived data** (both tabular and non-tabular) lives in **`s3://mg-ai-uw1-hyperpod-datasets`**. Path layout separates the two under each project's top-level prefix (Unity Catalog external locations vs. raw media-style folders).

- **Tabular data**: A path prefix such as `/.../__unity_catalog/*` or the `__unitystorage/**` branches below is wired as a Databricks Unity Catalog External Location.  
- **Non-tabular data**: Separate folder convention (e.g. `/.../images/*`); project examples below.

```
# example:
/data/mgai/tinder/recs/__unitystorage/** -> tabular
/data/mgai/tinder/recs/images/**         -> non-tabular
/data/mgai/tinder/tns/__unitystorage/**  -> tabular
/data/mgai/tinder/tns/images/**          -> non-tabular
/data/mgai/pairs/recs/__unitystorage/**  -> tabular
/data/mgai/pairs/recs/images/**          -> non-tabular
```

**DLP system storage** uses **`s3://mg-ai-uw1-s3-system-inventory`**. That bucket is for DLP system and inventory artifacts (S3 inventory snapshots, deletion manifests, pipeline control tables); it is **not** where HyperPod tabular or non-tabular datasets are stored.

# Authentication & Credential Management

### 1\. Delta Sharing (Tinder \-\> MG AI, Closed Event Tables Only)

Delta Sharing is used **exclusively** for receiving the 4 closed event tables required by the DLP pipeline. No other Tinder data (raw S3 data, general Delta tables) is shared via this mechanism.

| Item | Details |
| :---- | :---- |
| Mechanism | Delta Sharing (open protocol) |
| Authentication | \[TBD: e.g., Sharing profile with bearer token (`*.share` profile file)\] |
| Credential storage | \[TBD: e.g., Databricks Secrets / AWS Secrets Manager\] |
| Credential rotation | \[TBD: rotation policy\] |
| Shared tables | 4 closed event tables only (see Brand Infra Integration section) with `WITH HISTORY` option |

### 2\. DLP Service Principal (Databricks)

All DLP pipeline jobs run under a dedicated **Databricks Service Principal**, not individual user credentials. This principal is the sole identity authorized to write to core DLP system tables and execute deletion operations.

| Item | Details |
| :---- | :---- |
| Service Principal Name | \[TBD: e.g., `sp-dlp-pipeline`\] |
| Workspace | MG AI Databricks (us-east-1) |
| Permissions | Owner of DLP system catalog/schema; `DELETE`, `UPDATE`, `VACUUM` on DLP-managed tables; `READ` on Delta Sharing sources |

### 3\. S3 Batch Operations IAM Role

S3 Batch Operations for non-tabular data deletion runs under a dedicated IAM role scoped to the target buckets.

| Item | Details |
| :---- | :---- |
| IAM Role | \[TBD: e.g., `arn:aws:iam::<account>:role/dlp-s3-batch-ops`\] |
| Allowed actions | `s3:DeleteObject`, `s3:GetObject` (for manifest read), `s3:CreateJob`, `s3:DescribeJob` |
| Resource scope | `arn:aws:s3:::mg-ai-uw1-hyperpod-datasets/*` |
| Credential type | \[TBD: STS temporary credentials (assumed by Databricks job cluster)\] |

# Access Control

The fundamental direction for access control is the **"Principle of Least Privilege (Deny by default)"**. Direct access to core DLP system tables and the original mapping tables is strictly limited to infrastructure administrators and the DLP Service Principal.

### 1\. Group and Permission Synchronization (Databricks & HyperPod)

User permissions are managed based on project-level groups and ACLs.

- **Databricks Groups**: Classified into project-level groups (e.g., MGAI Tinder Recs, MGAI Tinder RnS, MGAI Aura, etc.), and managed through IaC ([mg-databricks](https://github.com/matchgroup/mg-databricks), [mgcore-terragrunt](https://github.com/matchgroup/mgcore-terragrunt)).  
- **HyperPod Groups**: Mapped to Linux groups (e.g., `mgai-aura`) for separate management. (Currently operating based on GitOps, planned to be synchronized via automated PR creation when Databricks permissions change in the future).

### 2\. S3 Access Control (Blocking Direct API Access)

- Direct access via AWS IAM is **blocked** for MG AI buckets (`s3://mg-ai-uw1-hyperpod-datasets`, `s3://mg-ai-uw1-s3-system-inventory`).  
- Only access via Databricks Unity Catalog External Location (for DBX Jobs, DBX Notebooks, etc.) or FSx (for HyperPod-running processes) is allowed.  
- The S3 bucket policy denies all non-allowed principals and enforces `aws:SecureTransport`.

### 3\. Detailed Access Policies by Data Type

- **Closed Object (User, Photo) Table**: Read-only, shared only with the minimum necessary personnel.  
- **Audit Log**: Restricted to infrastructure administrators and managers.  
- **Tabular Derived Data** (created by MLE): Controlled via Unity Catalog tags, Upstream ABAC inheritance, Column Masking, and Baseline Grants.  
- **Non-tabular Derived Data** (created by MLE): Separated by project groups and directory ACLs. Cross-validation of allowed groups and S3 Prefixes upon data import requests.  
- **Pseudonymization Mapping Table (`safe_id_mappings`, uid \<-\> safe\_id map):** Engineers have only **read** access through UDF lookups (`core.dlp.udf_lookup_safe_id`). Direct table **write/delete** permissions are restricted to the DLP Service Principal and a limited number of admins. No direct `INSERT` access is granted to MLE pipelines.

# Deletion Process and Completion Verification

Deletion requests are strictly tracked by status values from reception to final verification. The system uses a **2-Phase Verification** architecture: individual sub-tasks are created per deletion target, and the parent event is only marked complete when all sub-tasks pass verification.

**Data Deletion Process**

1. **Deletion Reception**: Registered in the `closed_objects` tracking table upon detecting account withdrawal/deletion events via Delta Sharing streaming.

2. **Sub-task Generation**: For each deletion event, individual `deletion_tasks` are created for each target (tabular data, non-tabular S3 objects, pseudonymization mappings).

3. **Tabular Data Deletion**: Target tables are identified via Unity Catalog tag scan (`dlp_method: row_delete`). `DELETE` \+ `VACUUM` is executed on each table.

4. **Pseudonymization Mapping Cleanup**: The user's entry in the single `safe_id_mappings` table is permanently deleted (`DELETE` \+ `VACUUM`), severing the link between the real identifier and the pseudonymized ID. This renders the remaining pseudonymized data mathematically irrecoverable. (Tables tagged with `dlp_method: pseudonymized` are tracked for monitoring purposes.)

5. **Non-tabular Data Deletion (S3)**: A manifest CSV is generated from the S3 inventory, and AWS S3 Batch Operations executes bulk `DeleteObject` calls.

6. **S3 Deletion Verification (Anti-Join)**: On the following day, the newly ingested S3 inventory snapshot is compared against the deletion target list via **Anti-Join**. If a target file no longer appears in the inventory (or only a Delete Marker remains), the deletion is verified. This approach avoids direct S3 API calls and provides audit-grade evidence.

7. **Status Rollup**: Once all sub-tasks for a parent event are `verified`, a single bulk `UPDATE` marks the parent `closed_objects` record as `deleted`.

*Note (SLA): The default SLA for deletion processing is 48 hours, but we plan to fine-tune this later based on actual measurement data.*

# Audit Logs

All DLP operations are recorded in a unified `audit_logs` table, differentiated by `action_type`:

| `action_type` | Description |
| :---- | :---- |
| `structured_delete` | Row deletion from Delta Tables \-- tabular data (DELETE \+ VACUUM) |
| `s3_batch_delete` | S3 Batch Operations execution and result |
| `cache_purge` | FSx cache and HyperPod local copy cleanup |
| `mapping_cleanup` | Pseudonymization mapping (`safe_id_mappings`) deletion |

Each log entry includes: `log_id`, `event_id` (FK to `closed_objects`), `action_type`, `status`, `details` (JSON metadata), `error_message`, and `created_at`.

Additionally, the system maintains:

- **`path_convention_violations`** table: Tracks S3 path convention violations detected by the daily inventory scan, including grace period and enforcement status.  
- **Pipeline control table**: Manages which tables are subject to the DLP pipeline (separate from audit logs).

Operational access to all audit tables is restricted to infrastructure administrators/managers only. Audit logs follow a structured JSON format (JSON Lines) for integration with centralized log collection systems (e.g., Datadog, Splunk).

# 🗑️ Stashed

# \[WIP\] Technical details

# MG AI DLP System Technical Specification

## 1\. Core Philosophy Recap

The MG AI DLP system does not define its own deletion rules. Instead, it receives **Account Closed** and **UGC Deleted** events from each brand (e.g., Tinder) and synchronizes/deletes derived data within the HyperPod environment accordingly.

Four core methodologies (Method A, B, C, and D) are applied based on data ownership and format:

| Data Type | Example | DLP Method |
| :---- | :---- | :---- |
| Data directly managed by brands (e.g., Tinder) | Tinder Raw S3 Data | A) Read-only mount with brand's S3. |
| Tabular data created by MG AI | User Feature Tables, Recommendation Rankings | B) Delete closed accounts records using Databricks/Delta lake. C) Pseudonymize user identifiers using Databricks. |
|  |  |  |
| Non-tabular data created by MG AI  | Cropped Profile Images, Audio | D) Delete files by path convention. |

## 2\. Infrastructure Setup

### 2.1 Infrastructure Management & Core Concepts

**IaC Management**: Managed via `matchgroup/cpts-Databricks` (Databricks) and `matchgroup/terra-aws-coe` (AWS) repositories to ensure consistency and auditability.

**Global Automated Pipeline**: A centralized engine running as a scheduled job within the `mg-ai` Databricks workspace. It handles Delta table purging, S3 inventory ingestion, deletion verification, mapping table cleanup, and Bad Path Registry enforcement.

**Security: Pseudonymization**: A secure mapping system replaces Real IDs with safe identifiers.

* **Flow**: `Real ID` ➡️ `SHA-256 Hash (hashid)` ➡️ `NanoID (safe_id, 21 chars)`.  
* **Security**: Uses NanoID for URL-safety and Z-Order/Bloom Filter indexing for high-performance lookups.  
* **Schema**: Includes `hashid` (PK), `brand`, `uid`, `user_number`, `safe_id`, and `created_at`.

### 2.2 Databricks Environment & Data Sharing

**Workspace & Connectivity**: Uses the `mg-ai` Databricks Workspace with **Delta Sharing** to mount external data (e.g., Tinder) without physical replication.

**Unstructured Data Governance**: External brand S3 buckets are mounted as **External Volumes** for centralized governance.

* **Restrictions**: ML Engineers are strictly prohibited from copying raw unstructured data into `mg-ai` internal buckets.  
* **Storage**: Processed data must follow **standard path naming conventions** for automated governance.

**Unity Catalog (UC) Structure (Reference)**:

```
Metastore (Enterprise Data Hub)
┃
┣━━ 📂 core (Company-wide common & operational data)
┃    ┣━━ 📁 dlp (Storage for DLP processing data & Mapping Tables)
┃    ┗━━ 📁 monitoring (Pipeline logs & quality metrics)
┃
┣━━ 📂 {project_name} (Project-specific isolated environments)
┃    ┣━━ 📁 bronze (Raw: Source data ingestion)
┃    ┣━━ 📁 silver (Refined: Cleansing & integration)
┃    ┗━━ 📁 gold (Aggregated: Analytics/Service-ready aggregation)
┃
┣━━ 📂 external (External sharing & source management)
┃    ┣━━ 📁 sharing_in (Delta Sharing received data)
┃    ┗━━ 📁 sources (Logical space for gathering external sources)
┃
┗━━ 📂 system (Databricks system management)
     ┗━━ 📁 information_schema (Metadata/Permission lookups)
```

---

### 2.3 AWS S3 Storage Infrastructure

**Storage Buckets**:

* `mg-ai-uw1-hyperpod-datasets`: Main bucket for model training (mounted to HyperPod).  
* `mg-ai-uw1-s3-system-inventory`: Dedicated bucket for S3 Inventory reports.

**Configuration**:

* **Versioning**: Enabled on all target buckets for recovery and auditability.  
* **Inventory Export**: Daily Parquet exports with `IncludedObjectVersions=All` to track all object versions and delete markers for complete physical deletion verification.

---

### 2.4 Automated Deletion & Verification Pipeline

**Structured Data Deletion**:

* **Targeting**: Tables must be tagged with `dlp: enabled` and include standard identifier columns.  
* **Physical Removal**: Daily jobs perform `DELETE` (logical) followed by `VACUUM` (physical). **VACUUM retention is set to 24 hours**.  
* **Mapping Purge**: Pseudonymization mapping is only purged after all associated data objects have reached `DELETED_VERIFIED` status.

**Unstructured Data Verification (AWS S3)**:

* **Ingestion**: Daily S3 Inventory reports are ingested using an **Auto Loader pipeline**.  
* **Verification**: A dedicated job compares requested deletions against the latest S3 Inventory.  
* **Confirmation**: Deletion is verified only when the object key (and all versions/markers for versioning buckets) is completely absent from the inventory.

**SLA & Incident Response**: If a deletion is not verified within **48 hours**, the system triggers an **on-call incident via incident.io**.

---

### 2.5 Bad Path Registry (Governance)

**Detection & Notification**:

* **Daily Scan**: Scans the HyperPod dataset bucket for path naming convention violations.  
* **Alerts**: Maps S3 prefixes to owners/Slack channels. Notifications are sent **every 4 hours** until resolved.

**Remediation**: Automatically performs a `Hard Delete` if the violation is not resolved within the grace period (`delete_after`).

## 3\. DLP Implementation Details \[WIP, tentative\]

### 3.1 Common Requirements

* Workspace: Use `mg-ai` DBX Workspace within MG Core.  
* Data Mounting: Tinder Databricks data (e.g., Closed account/UGC tables) is mounted to UC via the Delta Shares Received section.  
* Cluster Policy: Apply instance types and tagging policies per usage (Query, General, Training, Inference).

### 3.2 Method A: Read-only mount with brand's S3

TBD

3.3 Method B: Structured Data Deletion (Row-level)

1. Closed Account Information Collection (Intake)  
   * Source: Tinder `account_backend_hard_delete`, `account_id_mappings` tables.  
   * Method: Periodic collection (6–24 hours, variable) configured as a View via SQL Warehouse permissions.  
2. Control Table Schema (`dlp_control.closed_objects`) Tracks deletion targets and task status.

| Column | Type | Description |
| ----- | ----- | ----- |
| `obj_id` | STRING | Unique identifier in DLP (e.g., O-001) |
| `user_id` / `user_num` | STRING | User identifier (at least one required) |
| `target_type` | STRING | `USER_ROOT` or `UGC` |
| `ugc_type` / `ugc_id` | STRING | Type (image, video, audio) and ID |
| `target_path` | STRING | Target S3 URI (for unstructured data) |
| `status` | STRING | `PENDING`, `COMPLETED`, `FAILED`, `NOT_FOUND` |
| `withdrawal_at` | TIMESTAMP | Event occurrence time at source |
| `escalated_at` | TIMESTAMP | Escalation time if task fails for \>48h |

3. Delta Table Cleanup Logic Queries `information_schema` to identify tables tagged with `dlp.enabled = true` and performs deletion.

```py
# (Example) Tag-based iterative deletion code
target_tables = spark.sql("SELECT table_catalog, table_schema, table_name FROM information_schema.tables WHERE ...")
for row in target_tables:
    full_name = f"{row.table_catalog}.{row.table_schema}.{row.table_name}"
    spark.sql(f"DELETE FROM {full_name} WHERE user_id IN (SELECT subject_id FROM dlp_control.closed_objects WHERE status='PENDING')")
    spark.sql(f"VACUUM {full_name} RETAIN 24 HOURS")
```

### 3.4 Method C: Pseudonymization

Applied to destroy personal information while preserving statistical value.

1. Random ID Generation: NanoID  
   * Chosen because it provides collision resistance equivalent to UUID v4 while being shorter. It offers better readability, fast generation, and flexible length adjustment.  
2. Mapping Table Schema (`mapping.id_pseudonym_map`)

| Column | Type | Description |
| ----- | ----- | ----- |
| `raw_id_hash` (PK) | STRING | SHA-256 hash of `raw_id` for indexing performance. |
| `raw_id` | STRING | Original identifier (encryption recommended). |
| `pseudo_id` | STRING | Pseudonym identifier generated via NanoID. |
| `created_at` | TIMESTAMP | Generation timestamp (for auditing). |

3. Pseudonymization UDF Logic  
   * Computes hash of input `raw_id` and queries the mapping table.  
   * If Mapping Exists: Returns the existing `pseudo_id`.  
   * If Mapping Missing: Generates a new NanoID, stores it in the mapping table, and returns it.  
4. Optimization Strategies  
   * Z-Order Indexing: Performed on `raw_id_hash` to optimize data layout and reduce search range.  
   * Bloom Filter Index: Checks for `raw_id` existence before reading files to prevent unnecessary I/O.

### 3.5 Method D: Unstructured Data Deletion (Path Convention)

Unstructured data (images, audio, etc.) follows strict path rules and uses an S3 Inventory-based indexing system for mass deletion performance.

1. Helper Library: `PathBuilder` Engineers must generate paths using this library:

```py
PathBuilder.build(
    project="tinder_recs",
    user_id="user_123",
    imageid="img_abc",
    ext="jpg"
) 
# Result -> /data/mgai/tinder_recs/uid=user_123__imageid=img_abc.jpg
```

2. S3 Inventory-based Indexing System Instead of scanning hundreds of millions of files, deletion candidates are managed via AWS S3 Inventory.  
   * Export: Daily Parquet export of `mg-ai-uw1-hyperpod-datasets` inventory to a system bucket (`mg-ai-s3-system-inventory`).  
   * Ingestion: Streaming ingestion via Auto Loader into a DBX Managed Table (`s3_inventory_daily`).  
   * Optimization: Partitioned by `brand`, `path`, and `uid` to improve query performance.  
3. Deletion Execution Process (S3 Batch Operations)  
   * Target Extraction: Joins `closed_objects` with the inventory table to extract specific S3 Keys.  
   * Mass Deletion: Calls S3 Batch Operations API to request parallel deletion of millions of objects.  
   * FSx Sync: Data deleted in S3 is automatically removed from HyperPod's FSx filesystem via DRA (Data Repository Association).  
4. Bad Path Registry and Watchdog  
   * Registry: Identifies non-compliant files and registers them as `ISOLATED` in the `Bad Path Registry` table.  
   * Watchdog: Notifies the owner and grants a 24-hour grace period. If no action is taken, the file is permanently deleted after recording audit logs.

## 4\. Operational Policy

### 4.1 Daily DLP Job Components

* Intake Job: Collects Closed events and stores them in `closed_objects`.  
* Worker Jobs (Parallel):  
  * Structured data: Performs Delta Table deletion and Vacuum.  
  * Pseudonymized data: Deletes rows from mapping tables.  
  * Unstructured data (files): Filters deletion targets via S3 Inventory and executes Batch Deletion.  
* Watchdog Jobs: Identifies path violations and enforces destruction.

### 4.2 Incident Response & Escalation

* Retries: Automatic retry up to 3 times for all job failures.  
* Escalation: Items not processed within 48 hours trigger an alert to the manager and an On-call notification.

### 4.3 Audit Log

All deletion attempts and results are recorded in the `closed_objects` table. Deletion evidence is managed using `completed`, `not_found`, and `failed` flags.

# \[outdated\] Tech Spec

# \[WIP\] MG AI HyperPod DLP Plan

## Technical Specifications

Author: [Jason Park](mailto:jason.park@match.com)  
Last Updated: Feb 25, 2026

# Overview

This document defines the **DLP (Data Lifecycle Policy)** standard for safely and efficiently securing data required for ML model training while protecting users' personal information.  
---

# TO-BE Architecture

![image6](../images/d5f2b55beed4.png)

---

# Background Knowledge

### DLP Triggers

When a **user account is closed (Account Closed)** or a **direct deletion request is made (UGC Deleted)**, the associated data enters a deletion grace period. After the grace period expires, DLP is triggered and the relevant files and data are deleted.

| User Type | Data Type | Retention Period |
| :---- | :---- | :---- |
| **Non-banned users** | UGC, lat/long | 90 days |
|  | Analytics data, Service usage log, Most event data, User attributes | 1 year |
|  | Customer care records, App download related data, Consent opt-ins & outs | 5 years |
|  | Purchase / refund history | 10 years |
| **Banned users** | UGC, lat/long | 2 years |
|  | Others | (same with Non-banned users) |

For full details, refer to the internal DLP policy page. ([doc1](https://drive.google.com/file/d/1ykMM_6DvgyWBSqGmZvMC52fH0HiQugVd/view), [doc2](https://drive.google.com/file/d/1lo5PFSb3W1xqAVMfdTsFS-yx8KTp9Dht/view))

### Data Classification

| Classification | Definition | Key Examples | Management Principle |
| :---- | :---- | :---- | :---- |
| **✅ DLP Subject** | Data that can identify individuals and must be deleted upon DLP expiration. | Original images, conversation logs, files derived from user IDs. | Delete within the retention period (90–365 days) after the triggering event. |
| **❌ DLP Non-Subject** | Data that cannot identify individuals and can be retained indefinitely. | **Pseudonymized (anonymized) data**, model weights, statistical metrics. | Permanent retention and asset management. |

**Pseudonymization:** The processing of personal data such that, without access to additional information that maps back to the original, the data cannot be attributed to a specific individual.

---

# DLP System Design

## DLP System Overview

| Data Classification | Definition | Key Examples | DLP Handling Options |
| :---- | :---- | :---- | :---- |
| **Case A. Brand Raw Data** | Data directly managed by brands (e.g., Tinder) \- exists in S3. | User data queryable in Tinder DBX; original profile images; data registered in UTP | 1\. Raw Data RO Mount |
| **Case B. Structured Derived Data** | Tabular data derived by MG AI from raw data for model training, etc. | Feature data created for model training | 1\. Delta Lake (`DELETE` & `VACUUM`) 2\. Pseudonymization |
| **Case C. Unstructured Derived Data** | Non-tabular data created by MG AI. | Preprocessed user profile images. | 1\. Path convention compliance |

## Case A: Brand Raw Data

This section covers the integration between the S3 raw storage and the high-speed training storage (FSx), and the approach for neutralizing data when DLP expires.

| Option | Approach | Pros | Cons |
| :---- | :---- | :---- | :---- |
| **1\. Raw Data RO Mount** | Mount S3 raw data to FSx with RO (Read-Only) permission enforced. | \- Zero prep time for training data. \- No changes needed in the training code. | \- Copying original files via `cp` cannot be tracked. |

### A-1. Raw Data RO Mount

- When mounting S3 raw data to FSx, set the permission to **RO (Read-Only)**.  
  - **Example:** When mounting the Tinder S3 bucket to HyperPod, set the `/data/tinder/*` path to RO to restrict write access for MLEs.  
- Files synced to FSx are read-only and cannot be modified.  
- Copy commands like `cp` are still available, so it is difficult to track if an operator arbitrarily copies raw data.  
- Derived data must be stored according to defined rules or stored with encryption so that future DLP processing is possible.

## Case B: Structured Derived Data

Structured derived data refers to data that **can be stored in Delta format** from DBX or other sources. This section covers DLP compliance methods when utilizing this data on the HyperPod cluster.

| Option | Approach | Pros | Cons |
| :---- | :---- | :---- | :---- |
| **1\. Delta Lake** | Delete user rows with a `DELETE` query, then physically remove files with `VACUUM`. | Can selectively delete specific user data rather than the entire table. | `VACUUM` consumes compute resources. |
| **2\. Pseudonymization** | Replace user IDs with virtual IDs for storage. | **Permanent retention.** The personal data link is severed, allowing continued use without deletion. | The virtual ID ↔ real ID mapping must be managed separately in a secure mapping table. |

**📚What is Delta format?**  
Delta format is a combination of `.parquet` files and transaction logs. Actual data is stored as standard parquet, but a `_delta_log` directory inside the folder records a history of changes. Think of it as a "version-controlled data table" — like Git, but for data.

**📚What to watch out for when using Delta format:**

- **Do not read `.parquet` files directly** — always use the `deltalake` library or `spark.read.format("delta")`.  
- **Do not arbitrarily `cp` Delta format files exported from DBX to another path.** Doing so will cause them to be excluded from DLP management jobs, resulting in a DLP violation.

### B-1. Delta Lake (`DELETE` & `VACUUM`)

When exporting data from DBX to an S3 bucket, use `delta` format. The exported Delta Table files are automatically synchronized to HyperPod and ready for immediate use.

When a user subject to deletion is identified, a regularly scheduled cleanup job deployed in DBX inspects the Delta Tables on HyperPod and deletes the rows belonging to that user.

When creating a Delta Table, rename UID columns to standardized column names that the job can recognize. Otherwise, the job cannot identify the table and the data will not be automatically cleaned. The four currently planned standard columns are listed below — only rename columns that correspond to these standard columns.

| Standard Column Name | Description |
| :---- | :---- |
| `uid` | User ID |
| `user_number` | User Number |
| `target_uid` | Target (Pair) User ID |
| `target_user_number` | Target (Pair) User Number |

⚠️Some datasets use a nested structure which contains user identifiable data inside JSON data. In this case, we recommend to make a flat-style data structure or to use B-2. Pseudonymization.

#### Example Code

⚠️ The example code below is not a finalized implementation and is provided to illustrate the general flow. Please use it as reference only.

**1\. Create and process a Delta Table in DBX**  
First, preprocess the required data using SQL or PySpark in DBX, then save it to S3 in Delta Lake format.

- **Storage format:** **Must use `delta` format** (avoid standalone Parquet).  
- **Storage path:** Save under `s3://mg-ai-uw1-hyperpod-datasets/data/mgai/{project}/*`. *(Example path; may change during development.)*  
- **Column renaming:** **Must rename** unique ID columns (used as PKs) **to `uid`** before export.

```py
from pyspark.sql import functions as F

# 1. Standardize column names (apply full DLP convention)
# - Subject user:  'uid', 'user_num'
# - Target (pair): 'target_uid', 'target_user_num'
# This lets the management job clearly know which columns to use for DELETE queries.

df_to_save = df_features \
    .withColumnRenamed("user_id", "uid") \
    .withColumnRenamed("user_number", "user_num") \
    .withColumnRenamed("partner_id", "target_uid") \
    .withColumnRenamed("partner_number", "target_user_num")  # Adjust to actual column names.

# 2. Save in Delta format
# Use the per-project standard S3 path defined in the guidelines.
df_to_save.write.format("delta") \
    .mode("overwrite") \
    .save("s3://mg-ai-uw1-hyperpod-datasets/data/mgai/sample-project/delta/pair_features")
```

**2\. Load data on HyperPod**  
Delta Tables stored in S3 are synced to HyperPod, so they can be used from HyperPod using the same `/data/mgai/{project}/*` path.

```py
from deltalake import DeltaTable
import pandas as pd

# Delta table path
table_path = "/data/mgai/project_name/user_features"

# Create a DeltaTable object
# References _delta_log to recognize only valid, up-to-date data
dt = DeltaTable(table_path)

# Convert to a Pandas DataFrame for use in training
df = dt.to_pandas()

print(f"Records loaded: {len(df)}")
```

### B-2. Pseudonymization (Optional)

Replace users' real identifiers (e.g., `hong@gmail.com`) with random strings (e.g., `user_abc123`) for storage. Since individuals can no longer be identified, the data may legally qualify as "statistical data" rather than "personal information."

During the DLP validity period, the mapping between real IDs and virtual IDs is kept in a **Mapping Table**, allowing MLEs to look up real IDs when needed. A virtual ID generated for a user is stored in the mapping table, and the same ID is reused for subsequent requests for the same user — ensuring a consistent virtual ID per user at all times.

When exporting data from DBX to S3, no special handling is required. However, if the data contains user-identifying IDs or personal information (e.g., email addresses), these must be pseudonymized before storage. If pseudonymization is not feasible, **B-1 must be used** to prevent DLP violations.

Pseudonymized tables are used normally. Because the mapping table stores the relationship between original and virtual IDs at the time of pseudonymization, if a real ID is needed for a specific user, it can be retrieved by querying the mapping table.

When a pseudonymized user triggers DLP, the system automatically removes the original ID ↔ virtual ID relationship from the mapping table. Once removed, restoring the original ID from the virtual ID is impossible, completing the DLP process. Unlike the traditional approach where the dataset becomes unavailable upon DLP trigger, **this approach allows the dataset to continue being used even after DLP is triggered**.

- **Recommended use case:** Consistent evaluation sets; long-term model performance tracking.  
- ⚠️ **Caution:** Other fields that can identify a user — such as phone numbers or email addresses — must not be left in the data. The system cannot automatically remove them.  
- **Note:** A utility function to look up `safe_uid` by `user_number` is planned.

#### Diagram

![image7](../images/cabd60826d6b.png)

#### Example Code

⚠️ The example code below is **not a finalized implementation** and is provided to illustrate the general flow. Please use it as reference only.

**1\. Pseudonymization in DBX (data preparation)**

Preprocess large-scale data in DBX to replace real IDs with virtual IDs (pseudonymize), and save the mapping information to the mapping table.

```py
# DBX (PySpark) environment
from pyspark.sql import functions as F
from mgai_dlp import IdentityClient

# 1. Load raw data (e.g., chat logs, gifting records, pair data)
raw_df = spark.read.table("prod_db.pair_activity_logs")

# 2. Initialize the pseudonymization client (connects to the mapping API)
id_client = IdentityClient(api_token="dbx-secure-token")

# 3. Define the pseudonymization UDF
# The same real ID always maps to the same virtual ID.
@F.udf(returnType="string")
def get_safe_uid(real_id):
    if real_id is None:
        return None
    # Returns an existing mapping if available, otherwise creates and saves a new one
    return id_client.get_or_create_safe_uid(real_id)

# 4. Apply pseudonymization to multiple columns and remove real IDs
# Both subject (user_id) and target (target_id) are replaced with virtual IDs.
pseudonymized_df = raw_df \
    .withColumn("safe_uid", get_safe_uid(F.col("user_id"))) \
    .withColumn("safe_target_id", get_safe_uid(F.col("target_id"))) \
    .drop("user_id", "target_id")  # Remove original PII columns immediately

# 5. Save pseudonymized data as a Delta table
# Pseudonymized data can be retained permanently — only the mapping table needs cleanup on user departure.
pseudonymized_df.write.format("delta") \
    .mode("overwrite") \
    .save("s3://mg-ai-uw1-hyperpod-datasets/data/mgai/project_a/pseudonymized_pair_features")
```

**2\. Retrieve the real ID of a pseudonymized user**

```py
from mgai_dlp import IdentityClient

# Look up real ID via API
id_client = IdentityClient(api_token="secure-token")

# Retrieve the real ID from a virtual ID (for analysis and debugging)
# May return None if the mapping has been deleted due to user departure.
real_id = id_client.reveal(safe_uid="user_abc123")

if real_id:
    print(f"Original User: {real_id}")
```

## Case C: Unstructured Derived Data

Derived data refers to data created by engineers from raw data for training purposes, such as **cropped images**. "Unstructured" here means data that cannot be deleted externally via a `DELETE` query (e.g., via Delta Lake) and synced locally.

| Option | Approach | Pros | Cons |
| :---- | :---- | :---- | :---- |
| **1\. Path convention compliance** | Enforce a required path structure when engineers create derived data; automatically delete files that violate the convention. | \- Data owner immediately identifiable from the path alone. \- On user departure, simply deleting the user's ID directory cleans everything up. | \- Engineers must strictly adhere to the storage path convention. \- Non-compliant files may be automatically deleted by the system. |

### C-1. Path Convention Compliance

- A folder and filename convention is defined in advance that engineers must follow when creating derived data.  
- When invalidating a specific user's data, simply deleting that user's folder ensures DLP compliance.  
  - Top-level categories are separated by folders; remaining metadata is embedded in the filename as `key=value` pairs.  
  - During DLP action, files are filtered and removed using regex.  
  - **Example:** On departure of user ID `12345`, run `rm -rf /data/mgai/*/uid=12345/` and `rm -rf /data/mgai/*/uid=12345_*/`  
- **Engineers must strictly follow these rules. Files that violate the rules are periodically deleted.** Before deletion, the system sends a Slack notification to the file creator; if no action is taken within 24 hours, the file is permanently deleted.  
  - **Example:** Creating `/data/mgai/my_project/invalid_path/data.pt` → Slack notification sent → Auto-deleted after 24 hours  
- While enforcing path conventions adds friction for engineers, a helper library (`mgai-dlp`) will be provided to ease compliance.  
- When DLP is triggered, the system deletes the user's and resource's folders and files from the S3 bucket. Deletions are also reflected in HyperPod, though with a slight delay.

#### Example rules.:

**Single-user data:**

```
/data/mgai/{project}/uid={uid}__imageid={imageid}.png
/data/mgai/{project}/user_number={user_number}__imageid={imageid}.png
```

**Multi-user data (e.g., swiper & swipee peered data)**  
Use multiple `uid` keys to indicate that multiple users are included.

```
/data/mgai/{project}/uid={uid}__uid={uid_2}__imageid={imageid}.png
```

**Single user as a directory with multiple images inside:**

```
/data/mgai/{project}/uid={uid}/image1__imageid={imageid}.png
/data/mgai/{project}/uid={uid}/image2__imageid={imageid}.png
/data/mgai/{project}/uid={uid}/image3__imageid={imageid}.png
```

**Pros:**

- Engineers can follow the path convention without a dedicated library, as long as they know the filename rules.  
- Adding new metadata does not require changes to the folder structure.

**Cons:**

- Search speed degrades as file count grows. This can be partially mitigated with fast search tools like [`fd`](https://github.com/sharkdp/fd) (a Rust-based `find` alternative — up to 10× faster for large directories).  
- Risk of training data loss if filename rules are not properly understood.  
- **255-character filename limit** restricts how much metadata can be embedded.  
- 

#### Diagram

![image8](../images/ce7d24c4b4b2.png)

#### Example Code (tentative)

💡 The `mgai-dlp` library is used to store and manage data at standardized paths.

**1\. Allocate a standard path and save data**

MLEs use the library to generate a path that includes user ID and resource ID, then save data to it.

```py
from mgai_dlp import PathManager, DataType
import torch

# 1. Generate a single-user data path
# Result: /data/mgai/my_project/u_12345/crop_01.pt
user_path = PathManager.get_path(
    project="my_project",
    user_id=12345,
    resource_id="crop_01",
    resource_type=DataType.IMAGE
)

# 2. Save data
tensor_data = torch.randn(100)
torch.save(tensor_data, user_path)
```

**2\. Automatic deletion of non-compliant data**

- Files that fall outside the designated path convention are periodically scanned by the system.  
- On violation, a Slack notification is sent; if not addressed within 24 hours, the file is permanently deleted.

# \[outdated\] Raw S3 mount

| Data directly managed by brands (e.g., Tinder) | A) Read-only mount with brand's S3. | Mounts brand-owned S3 buckets to the training environment with read-only permissions to prevent arbitrary modification. This relies on the original brand owner to handle the actual data disposal and compliance. |
| :---- | :---- | :---- |

## Method A) Read-only mount with brand's S3.

*This method is for dealing with data directly managed by brands.*

S3 raw data is mounted to the MG AI's S3 with strictly enforced Read-Only permissions. This ensures that engineers cannot modify or delete original brand data within the HyperPod environment, leaving the primary data lifecycle management to the respective brand's own compliance systems.

Note that we do not sync all files from Tinder to MG AI S3, but only **sync the S3 paths required for the project.** Also, we will manage user-level permissions for each data directory, and we'll track change history of access user list via Git.

Example config:

```
- project: Tinder Recs x MG AI
  - source_path: s3://tinder-ml-recs/datasets/*
  - target_path: s3://mgai/tinder/recs/datasets/*
  - accessible_user_list: [...]

- project: Tinder Photo Coaching x MG AI
  - source_path: s3://tinder-profile/user-photos/*
  - target_path: s3://mgai/tinder/photo-coaching/user-photos/*
  - accessible_user_list: [...]
```








