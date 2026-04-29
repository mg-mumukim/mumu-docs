# DLP System: MLE Usage Scenarios

> **Source memo**
> - `note/source/2026-04-23-dlp-system-design-mgai-docs.md` — DLP system design docs (15 documents from mgai-docs repo)
> - Slack `#mgai-data-governance-project` — Owen/Jason DLP feedback thread (2026-03-30)
> - Slack `#mgai-pairs` — Owen/Groot/Harvey TTL discussion (2026-04-10)
> - Slack `#mgai-aura` — Link Lee MLE DLP interview notes (2026-01-15); Mumu/Juniper PBD + DLP retention discussion (2026-04-08)
> - Slack `#analytics` — Tinder Data Eng DLP anonymization announcements (2025-05 through 2025-06)
> - Slack `#mgai-general` — Diego uid data quality issue from DLP logic divergence (2026-03-11)
> - Slack `#ml-team-recs` — Lichao/Shuaiji/Yu-Hsuan Databricks Volumes + DLP compliance discussion (2025-11-21)

## Purpose

This document enumerates scenarios where the Data Lifecycle Policy (DLP) system intersects with an ML engineer's (MLE's) day-to-day work. Each scenario is written as a user story at a technical level, covering the infrastructure stack described in the system design: Databricks (Unity Catalog, Delta Tables, Delta Sharing), AWS S3, SageMaker HyperPod with FSx for Lustre, and the `mgai-dlp` Python library.

Scenarios are organized along two axes to achieve MECE coverage:

1. **DLP track** — event-driven deletion, TTL-driven deletion, compliance/violation enforcement, or pseudonymization.
2. **MLE activity phase** — project setup, data ingestion, preprocessing, training, evaluation, storage management, or incident response.

---

## A. Project Setup and Onboarding

### S01. Tag a new Delta Table for row-level deletion

As an MLE creating a new feature table (e.g., `tinder.recsys.user_embeddings`) in the MG AI Databricks workspace, I add the Unity Catalog tag `dlp_method: row_delete` and include a `brand` column so that the DLP pipeline can automatically discover the table, generate deletion sub-tasks, and execute row-level `DELETE` when a user closure event expires.

- **DLP track:** event-driven (structured deletion)
- **Key interaction:** MLE applies UC tags at table creation time; the `execute_row_deletes` task picks up the table from `information_schema.table_tags` in subsequent runs.

### S02. Tag a new Delta Table for TTL-based expiry

As an MLE creating a daily event log table (e.g., `tinder.aura.coaching_events`) in Databricks, I apply the tags `dlp_method: ttl`, `dlp_ttl_days: 90`, and `dlp_ttl_column: created_at` so that the `execute_structured_ttl` task automatically deletes rows older than 90 days without requiring per-event deletion requests.

- **DLP track:** TTL-driven (structured deletion)
- **Key interaction:** MLE configures retention at table creation; the daily TTL job handles cleanup. Similar to the Pairs team's manual `DELETE WHERE event_date < (run_date - 89d)` pattern, but automated via DLP.

### S03. Mark a table as explicitly excluded from DLP deletion

As an MLE maintaining a reference table (e.g., `tinder.default.country_codes`) that contains no user-identifiable data, I add the tag `dlp_method: none` so that the DLP scan explicitly skips this table and does not log spurious warnings about missing identifier columns.

- **DLP track:** explicit exclusion
- **Key interaction:** MLE applies the `none` tag; `execute_row_deletes` and `execute_structured_ttl` both skip tables with this tag.

### S04. Onboard a new brand to the DLP system

As an MLE responsible for the Pairs data pipeline, I need to register Pairs-specific source tables (e.g., Pairs closed-account events) as Delta Sharing sources for the `ingest_closed_objects` streaming job, and ensure all downstream Pairs Delta Tables carry the appropriate `dlp_method` tags with a `brand = 'pairs'` column so the DLP pipeline can isolate Pairs deletions from Tinder deletions.

- **DLP track:** event-driven (cross-brand setup)
- **Key interaction:** MLE coordinates with the DLP system owner to configure Delta Sharing for new brand sources and validates the `brand` column exists in all tagged tables.

---

## B. Data Ingestion and Preprocessing

### S05. Download raw images from Tinder S3 and save to MG AI S3 with compliant paths

As an MLE building a photo moderation model, I download profile images from the Tinder S3 bucket via the S3 API, preprocess them, and save the results to `s3://mg-ai-uw1-hyperpod-datasets/data/mgai/photo_mod/uid={uid}__media_type=photo__media_id={media_id}.jpg` using the `mgai-dlp` `PathBuilder` to generate DLP-compliant filenames. When a user's account is closed, the DLP pipeline can locate and delete these files via filename token matching.

- **DLP track:** event-driven (unstructured deletion)
- **Key interaction:** MLE uses `PathBuilder` to construct S3 paths with `uid` and `media_id` tokens; the `generate_unstructured_manifest` task later matches these tokens against `closed_objects`.

### S06. Build a feature Parquet dataset with user identifiers embedded in filenames

As an MLE creating training features from Databricks tables, I export Parquet files to `s3://mg-ai-uw1-hyperpod-datasets/data/mgai/recsys_features/uid={uid}__user_number={user_number}.parquet`. The double-underscore delimited tokens in the filename enable the DLP manifest generation SQL to decompose and exact-match identifiers, preventing partial-match false positives (e.g., `uid=USR_A` does not accidentally match `uid=USR_A_PROD`).

- **DLP track:** event-driven (unstructured deletion)
- **Key interaction:** Correct path convention is critical; the filename tokenization logic relies on the `__` delimiter and `key=value` format.

### S07. Validate S3 paths before uploading preprocessed data

As an MLE about to upload a batch of preprocessed files to S3, I call `mgai_dlp.validate_dlp_path()` on each target path in my preprocessing script to catch missing identifiers (e.g., a path without `uid=`) before the files land in S3. This avoids triggering a violation alert from `scan_inventory_violations` the next day.

- **DLP track:** compliance (pre-upload validation)
- **Key interaction:** MLE integrates path validation into the data pipeline; invalid paths are rejected locally rather than discovered by the daily scan.

---

## C. Pseudonymization and Data Preservation

### S08. Pseudonymize a training dataset to preserve it after user deletion

As an MLE maintaining a long-term evaluation dataset, I replace all `uid` columns with `safe_uid` by calling `core.dlp.udf_assign_safe_id('tinder', uid)` on the distinct user list and then broadcast-joining the result back to the full dataset. Once the pseudonymization mapping is recorded in `safe_id_mappings`, the original `uid` is dropped. Even after the user's account closure event triggers deletion, this dataset remains usable because the `safe_uid` cannot be traced back to the original user.

- **DLP track:** pseudonymization
- **Key interaction:** MLE calls the UDF to get or create a `safe_id`, then drops the raw identifier. The `destroy_pseudo_mappings` task later deletes the mapping row, making the pseudonymization irreversible.

### S09. Look up an existing safe_id without creating a new mapping

As an MLE joining a pseudonymized evaluation set with a new batch of inference results, I call `core.dlp.udf_lookup_safe_id('tinder', uid)` to retrieve the existing `safe_id` for each user. The read-only lookup returns `NULL` for users not yet mapped, allowing me to decide whether to assign a new mapping or skip those users.

- **DLP track:** pseudonymization (read-only)
- **Key interaction:** MLE uses the read-only UDF to avoid unintended side effects (no new rows in `safe_id_mappings`).

### S10. Tag a table as pseudonymized so DLP destroys mappings on deletion

As an MLE who has pseudonymized a model evaluation table, I add the tag `dlp_method: pseudonymized` so that when a user closure event triggers, the `destroy_pseudo_mappings` task scans this tag, finds the corresponding rows in `safe_id_mappings`, and deletes the mapping. The pseudonymized data in the table itself is not deleted — it is rendered permanently anonymous.

- **DLP track:** event-driven (pseudonymization mapping removal)
- **Key interaction:** MLE tags the table; the DLP pipeline interprets this tag as "delete the mapping, not the data."

---

## D. Model Training

### S11. Train on HyperPod with DLP-compliant data cached on FSx for Lustre

As an MLE running a training job on SageMaker HyperPod, I read preprocessed data from the local FSx mount (`/data/mgai/photo_mod/...`). The files in this cache are synced from `s3://mg-ai-uw1-hyperpod-datasets` via DRA (Data Repository Association). When the DLP pipeline deletes files from S3, the FSx cache eventually reflects the deletion through the sync mechanism. I do not need to manually purge the cache, but I need to be aware that files deleted from S3 may still be readable from the FSx cache until the next sync cycle.

- **DLP track:** event-driven (unstructured deletion, cache propagation)
- **Key interaction:** MLE relies on DRA auto-sync; stale cache entries are a known latency window, not a compliance gap, because the source of truth (S3) is already clean.

### S12. Store model checkpoint files that contain no user identifiers

As an MLE saving PyTorch model checkpoints to `s3://mg-ai-uw1-hyperpod-datasets/data/mgai/recsys_model/model_weights/epoch_10.pt`, I note that model weight files do not contain user identifiers. The built-in `ALLOW_EXCEPTION` for the `*model_weights*` path pattern in `s3_rule_registry` ensures these files are not flagged as violations by `scan_inventory_violations`, even though they lack `uid=` tokens in the filename.

- **DLP track:** compliance (built-in exception)
- **Key interaction:** MLE places model artifacts under a path matching the `*model_weights*` pattern; no additional action required.

### S13. Save shared multi-user data files during training

As an MLE creating a training dataset of user-pair interactions (e.g., match pairs for a recommendation model), I save files with multiple user identifiers: `uid={uid1}__uid={uid2}__interaction=match.parquet`. Under the DLP shared-file policy, if either `uid1` or `uid2` triggers a deletion event, the entire file is deleted because the manifest join checks `array_contains(p.uids, t.uid)` — a match on any element is sufficient.

- **DLP track:** event-driven (unstructured deletion, shared files)
- **Key interaction:** MLE must understand that shared files are deleted if any referenced user is deleted. For long-term preservation, pseudonymization (S08) is the alternative.

---

## E. Evaluation and Serving

### S14. Run offline evaluation using pseudonymized historical data

As an MLE computing model metrics on a pseudonymized evaluation set, I join the evaluation table (keyed on `safe_uid`) with the latest model scores. Because the real `uid` was replaced and the mapping was destroyed after the user's deletion event, the evaluation set is not affected by user churn. Model performance trends remain stable across evaluation windows.

- **DLP track:** pseudonymization (post-mapping-destruction)
- **Key interaction:** MLE benefits from pseudonymization — the dataset survives user deletion cycles and supports long-term metric tracking.

### S15. Generate and store inference results that reference user identifiers

As an MLE running batch inference and writing results to a Delta Table (e.g., `tinder.recsys.daily_scores` with a `uid` column), I ensure the table has the `dlp_method: row_delete` tag. When a user closure event fires, the DLP pipeline deletes the corresponding rows. I also set `dlp_ttl_days: 30` and `dlp_ttl_column: scored_at` so that stale scores are automatically purged regardless of deletion events.

- **DLP track:** event-driven + TTL-driven (dual policy on one table)
- **Key interaction:** A table can participate in both event-driven and TTL-driven deletion simultaneously, each governed by its own tag.

[!NOTE] The source design document does not explicitly describe dual-tag scenarios (both `row_delete` and `ttl` on the same table). This scenario is plausible given the tag-based architecture but should be confirmed with the DLP system owner.

---

## F. Storage Management and TTL

### S16. Register a TTL policy for temporary experiment data on S3

As an MLE running short-lived experiments, I save intermediate results to `s3://mg-ai-uw1-hyperpod-datasets/data/tinder/experiments/exp_2026Q2/`. To avoid manual cleanup, I register a TTL rule in `s3_rule_registry`:

```sql
INSERT INTO core.dlp.s3_rule_registry (bucket_name, path_prefix, rule_type, ttl_days, description)
VALUES ('mg-ai-uw1-hyperpod-datasets', '/data/tinder/experiments/exp_2026Q2/', 'TTL', 30, 'Q2 experiment scratch data');
```

The `sync_s3_lifecycle_policies` job translates this into an S3 Lifecycle rule, and AWS auto-expires the objects after 30 days. The DLP pipeline verifies the expiration via the next inventory snapshot.

- **DLP track:** TTL-driven (unstructured deletion)
- **Key interaction:** MLE self-serves TTL registration; no coordination with the DLP system owner required.

### S17. Register an ALLOW_EXCEPTION for anonymized aggregate data

As an MLE storing pre-aggregated, anonymized statistics (e.g., cohort-level engagement metrics with no user identifiers), I register an exception in `s3_rule_registry`:

```sql
INSERT INTO core.dlp.s3_rule_registry (bucket_name, path_prefix, rule_type, ttl_days, description)
VALUES ('mg-ai-uw1-hyperpod-datasets', '/data/mgai/cohort_stats/', 'ALLOW_EXCEPTION', NULL, 'Anonymized aggregate metrics — no user identifiers');
```

The `scan_inventory_violations` job skips this path during its daily compliance check, preventing false-positive violation alerts.

- **DLP track:** compliance (custom exception)
- **Key interaction:** MLE adds a registry row; the violation scanner respects the exception from the next scan cycle.

---

## G. Compliance and Violation Response

### S18. Receive a Slack violation alert and fix the path within 24 hours

As an MLE who accidentally saved a file without proper identifiers (e.g., `/data/mgai/my_project/temp_output.csv`), I receive a Slack alert from `send_violation_alerts` identifying the non-compliant path. I have 24 hours to either move the file to a compliant path (adding `uid=` tokens) or delete it. If I fix the file before the next `check_grace_period_expiry` run, the violation status transitions to `RESOLVED_BY_USER`.

- **DLP track:** compliance (violation response)
- **Key interaction:** MLE acts on the Slack notification; the system checks the latest S3 inventory snapshot to confirm the fix.

### S19. System force-deletes a file after grace period expiry

As an MLE who did not respond to a violation alert within 24 hours, the DLP system transitions the violation to the enforcement path. `check_grace_period_expiry` marks the file as deadline-exceeded, `generate_unstructured_manifest` includes it in the next S3 Batch Operations manifest, and `execute_unstructured_delete` permanently removes the file. The violation status becomes `FORCE_DELETED`.

- **DLP track:** compliance (enforcement)
- **Key interaction:** MLE loses the file; the audit log records the forced deletion with `action_type: force_delete`.

### S20. Migrate legacy data to comply with new path conventions

As an MLE with existing data stored under an old directory structure (e.g., `/data/mgai/old_project/user_123/features.parquet` — identifier in the directory path, not in the filename), I need to reorganize files to the new convention (`uid=123__features.parquet`) before the DLP system flags them as violations. I write a migration script that reads the old paths, extracts identifiers, reconstructs compliant filenames using `PathBuilder`, copies the files, and then deletes the originals.

- **DLP track:** compliance (proactive migration)
- **Key interaction:** MLE proactively migrates before the scanner detects violations; the tokenization logic requires identifiers in the filename, not in directory segments.

---

## H. Incident Response and Auditing

### S21. Investigate a failed deletion event via audit logs

As an MLE or on-call engineer investigating why a specific user's data was not fully deleted (the `closed_objects` status is `failed`), I query `audit_logs` filtered by `event_id` and `status = 'failed'` to identify which sub-task failed. The `error_message` field reveals the root cause — for example, `AnalysisException: [TABLE_OR_VIEW_NOT_FOUND]` indicating the target table was dropped before the deletion task ran. I then coordinate with the DLP system owner to resolve the failure and retry.

- **DLP track:** event-driven (failure investigation)
- **Key interaction:** MLE queries the flat audit schema; the 1-row-per-target design makes it straightforward to pinpoint which specific table or S3 object caused the failure.

### S22. Discover that training data references a deleted user due to cache lag

As an MLE who notices unexpected `NULL` join results during model training on HyperPod, I suspect the training dataset still references a user whose data was recently deleted from Databricks. I check `closed_objects` to confirm the user's deletion status is `deleted`, then verify whether the S3 files were removed by querying `s3_bucket_objects` for the latest inventory. The S3 files are gone, but the FSx cache still serves the stale copy. I trigger a manual cache refresh or wait for the next DRA sync cycle.

- **DLP track:** event-driven (cache consistency)
- **Key interaction:** MLE uses DLP system tables as a diagnostic tool; the issue is infrastructure lag (FSx sync delay), not a DLP pipeline failure.

### S23. Handle a data quality incident caused by DLP logic divergence

As an MLE who discovers missing user records in `tinder_members_uid` after a platform migration (e.g., Redshift to Databricks), I investigate and find that DLP anonymization logic was applied differently during the migration, causing records to be filtered out instead of anonymized. I coordinate with the Data Engineering team to backfill the affected records and verify that the DLP pipeline's deletion logic is aligned across old and new systems.

- **DLP track:** event-driven (migration edge case)
- **Key interaction:** MLE discovers the issue through downstream data quality symptoms; root cause is a mismatch between legacy and current DLP implementations. This mirrors the real incident where 7K users were affected by a DLP logic divergence during a Databricks migration.

---

## Coverage Matrix

| Scenario | DLP Track | Data Type | MLE Phase |
| --- | --- | --- | --- |
| S01 | Event-driven | Structured | Project setup |
| S02 | TTL-driven | Structured | Project setup |
| S03 | Exclusion | Structured | Project setup |
| S04 | Event-driven | Cross-type | Onboarding |
| S05 | Event-driven | Unstructured | Ingestion |
| S06 | Event-driven | Unstructured | Preprocessing |
| S07 | Compliance | Unstructured | Preprocessing |
| S08 | Pseudonymization | Structured | Preprocessing |
| S09 | Pseudonymization | Structured | Preprocessing |
| S10 | Event-driven | Structured | Project setup |
| S11 | Event-driven | Unstructured | Training |
| S12 | Compliance | Unstructured | Training |
| S13 | Event-driven | Unstructured | Training |
| S14 | Pseudonymization | Structured | Evaluation |
| S15 | Event-driven + TTL | Structured | Evaluation |
| S16 | TTL-driven | Unstructured | Storage mgmt |
| S17 | Compliance | Unstructured | Storage mgmt |
| S18 | Compliance | Unstructured | Violation response |
| S19 | Compliance | Unstructured | Violation response |
| S20 | Compliance | Unstructured | Migration |
| S21 | Event-driven | Cross-type | Incident response |
| S22 | Event-driven | Unstructured | Incident response |
| S23 | Event-driven | Structured | Incident response |
