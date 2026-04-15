# OPaaS Sunset: Shutdown Guide

## Sources

- Notion: "OPaaS Deployment & Infrastructure" — https://www.notion.so/1373e5a379b34587ac21be7c6dd87496
- Notion: "[Product] OPaaS API Server" — https://www.notion.so/f7fae196259a4696b7916906d853b7bd
- Notion: "[Product] OPaaS Console" — https://www.notion.so/8a6a27344f534de9838b7ca010faa3ae
- Notion: "OPaaS (Central AI Platform)" — https://www.notion.so/1f6e373228e74b39833d47ed82a4724d
- Google Doc: "MG AI SRE/DevOps Task Ideation (Nov 2025 ver.)" by Owen Lee — https://docs.google.com/document/d/1qVRBDzzlWQd0ocHiXE0wJewTSFiioB_Pgys5Bbhuan4/edit
- Slack: #mgai-opaas (Hyperconnect workspace), #mgai_hinge_photo_picker_dev, #mgai-dev (Tinder workspace)

---

## Conclusion

OPaaS is ready to shut down. As of 3/31/25, Hinge disabled Photo Finder (the only production use of OPaaS); the Nov 2025 SRE task document (Owen Lee) confirms traffic is 0 and no brands are using the SDK. Hinge's SDK may still be compiled into the app binary but is inactive via feature flag — this is not a blocker for server-side shutdown.

**Shutdown must be executed in the section order below.** Sections 1–2 are prerequisite coordination. Sections 3–5 take down live services. Sections 6–16 clean up persistent resources. Destructive steps (database deletion, S3 deletion) must not be executed until the dependent k8s services are confirmed stopped.

Two items require external coordination before proceeding:
1. Cloudflare CDN rules and Route53 DNS — MG AI cannot access HPCNT Cloudflare (SSO restriction). Contact Dan Kim (HPCNT Infra) for removal.
2. Event Gateway — used by OPaaS SDK for usage logging, provided by HPCNT Core Platform Team. Confirm whether decommission or consumer removal is needed.

---

## Ordered Shutdown Checklist

### Section 1 — Confirm zero traffic and coordinate with Hinge

- [ ] Verify zero traffic in Grafana dashboards for all 3 environments (prod, beta, qa). Dashboard IDs in Evidence section.
- [ ] Confirm with Hinge (Kevin Sun, Doug Galante) that Photo Finder feature flag will remain off permanently and SDK will be removed in a future app release.
- [ ] Confirm with ASL (Art & Science Lab / Tinder) that no OPaaS SDK usage is active. ASL was a listed customer alongside Hinge.

### Section 2 — Disable automated deployments (CI/CD)

Disable before scaling down k8s to prevent accidental re-deploy.

- [ ] Disable GitHub Actions in `hyperconnect/opaas-backend` (`git.dev.hpcnt.com`): deactivate or delete `ci.yml` and `cd.yml` workflows.
- [ ] Disable GitHub Actions in `hyperconnect/opaas-frontend` (`git.dev.hpcnt.com`): same.
- [ ] Remove OPaaS entries from deploy-center (`git.dev.hpcnt.com/hyperconnect/deploy-center`) branches: `prod/main`, `beta/main`, `qa/main`.

### Section 3 — Scale down k8s deployments (QA → Beta → Prod)

All 3 deployments per namespace: `opaas-admin-api-prod`, `opaas-config-api-prod`, `opaas-frontend-prod`.

- [ ] **QA**: scale to 0 in namespace `opaas`, cluster `kube-dev-uw2-ai-platform-hpcnt-com`.
- [ ] **Beta**: scale to 0 in namespace `beta-opaas`, cluster `kube-prod-uw2-ai-platform-hpcnt-com`.
- [ ] **Prod**: scale to 0 in namespace `opaas`, cluster `kube-prod-uw2-ai-platform-hpcnt-com`.

### Section 4 — CDN and DNS (requires HPCNT devops)

MG AI cannot access HPCNT Cloudflare due to SSO restrictions. Request Dan Kim (HPCNT Infra) to perform these steps.

- [ ] Remove Cloudflare CDN cache rule for Config API (1 min TTL): rule ID `ed149e9e21b1485abd10964cfee76df1`, zone `59b61dfdaf996df7c8d763a0b7babd05` (hce.io). Dashboard: https://dash.cloudflare.com/59b61dfdaf996df7c8d763a0b7babd05/hce.io/caching/cache-rules
- [ ] Remove Cloudflare CDN cache rule for Model Download (1 day TTL): rule ID `51213d061f8d4d729f969b38c23b183c`.
- [ ] Remove Route53 DNS records for `opaas-admin` in hosted zone `Z0961120PZRFZ6FMZ1JZ`.

### Section 5 — Okta

Beta was migrated from HCE Okta to MatchGroup Okta on 3/26/25 (Parker Cho). Prod migration status is unconfirmed — check both tenants.

- [ ] Remove OPaaS application from HCE Okta (production console auth).
- [ ] Remove OPaaS application from MatchGroup Okta (beta console).

### Section 6 — Databases (backup first, then delete)

Snapshot all Aurora instances before deletion. DB spec: MySQL 8.0.

- [ ] Snapshot QA Aurora: `ai-platform-qa-opaas.cluster-cx42umccgb7k.us-west-2.rds.amazonaws.com:3306/opaas`
- [ ] Snapshot Beta Aurora: `ai-platform-beta-opaas.cluster-cvaecmfytoiu.us-west-2.rds.amazonaws.com:3306/opaas`
- [ ] Snapshot Prod Aurora: `ai-platform-production-opaas.cluster-cvaecmfytoiu.us-west-2.rds.amazonaws.com:3306/opaas`
- [ ] Delete QA Aurora instance.
- [ ] Delete Beta Aurora instance.
- [ ] Delete Prod Aurora instance.

### Section 7 — S3 Buckets (archive or delete)

6 buckets total. Consider archiving to Glacier before full deletion.

- [ ] `ai-platform-production-opaas-sdk-main-origin` (prod, SDK files)
- [ ] `ai-platform-production-opaas-model-main-origin` (prod, ML model files)
- [ ] `ai-platform-beta-opaas-sdk-main-origin` (beta, SDK files)
- [ ] `ai-platform-beta-opaas-model-main-origin` (beta, ML model files)
- [ ] `ai-platform-qa-opaas-sdk-main-origin` (qa, SDK files)
- [ ] `ai-platform-qa-opaas-model-main-origin` (qa, ML model files)

### Section 8 — Vault Secrets

Backend secrets are injected at pod startup (`kv-kube`). Frontend secrets are injected at Docker build time (`kv-build`). Delete after services are confirmed stopped.

- [ ] Delete `kv-kube/kube-prod-uw2-ai-platform-hpcnt-com/opaas/opaas-admin-api-prod-secret` — https://vault.kube-prod-an1.hpcnt.com/ui/vault/secrets/kv-kube/show/kube-prod-uw2-ai-platform-hpcnt-com/opaas/opaas-admin-api-prod-secret
- [ ] Delete `kv-kube/kube-prod-uw2-ai-platform-hpcnt-com/opaas/opaas-config-api-prod-secret`
- [ ] Delete `kv-kube/kube-dev-uw2-ai-platform-hpcnt-com/opaas/opaas-admin-api-qa-secret`
- [ ] Delete `kv-kube/kube-dev-uw2-ai-platform-hpcnt-com/opaas/opaas-config-api-qa-secret`
- [ ] Delete `kv-build/opaas-frontend/opaas-frontend/main/qa` — https://vault.kube-prod-an1.hpcnt.com/ui/vault/secrets/kv-build/show/opaas-frontend/opaas-frontend/main/qa
- [ ] Delete `kv-build/opaas-frontend/opaas-frontend/main/beta`
- [ ] Delete `kv-build/opaas-frontend/opaas-frontend/main/prod`

### Section 9 — Event Gateway

OPaaS SDK sends usage logs to HPCNT Core Platform's Event Gateway. Traffic is already 0.

- [ ] Confirm with HPCNT Core Platform team whether the Event Gateway instance is OPaaS-exclusive or shared with other services.
- [ ] If exclusive: decommission Event Gateway instance.
- [ ] If shared: remove OPaaS consumer configuration from Event Gateway.

### Section 10 — Monitoring and alerting

- [ ] Archive or delete 6 Grafana dashboards (IDs in Evidence section). Base URL: `https://grafana.kube-prod-an1.hpcnt.com`
- [ ] Archive or delete Kibana APM services `platform-opaas-admin-api` and `platform-opaas-config-api`. Base URL: `https://kibana-sso.kube-prod-an1.hpcnt.com`
- [ ] Delete Sentry prod project ID 630 at `https://sentry.svc.hpcnt.com/organizations/hpcnt/projects/opaas-backend/`
- [ ] Delete Sentry dev project ID 464 at `https://sentry-dev.svc.hpcnt.com/organizations/hpcnt/projects/opaas-backend/`
- [ ] Archive or delete Slack channel `#opaas_oncall`.

### Section 11 — Databricks

- [ ] Archive table `opaas_prod.b__kafka.backend_platform_metric_in_house_app_opaas__view` in `data-infra-mg-ai-uw2` workspace (https://dbc-d7b34265-86cb.cloud.databricks.com).
- [ ] Evaluate whether the `data-infra-mg-ai-uw2` workspace can be fully deprecated (per Owen Lee's SRE doc — check for any other non-OPaaS workloads).

### Section 12 — GCP / Firebase PoC resources

These are demo/PoC resources, not the main HPCNT k8s infra. Contact Jason Park for GCP admin access.

- [ ] Delete Cloud Run service `collectfeedback` in region `us-central1`, project `ondevice-platform-poc`.
- [ ] Delete Firebase project `ondevice-platform-poc` (or verify it was already migrated/removed; earlier Slack suggests it may have been abandoned in favor of `mgai-dev`).
- [ ] Verify Firebase project `mgai-dev` — delete OPaaS-related apps (`mgai.tailor.playground` iOS app) if the project is not used by other workloads.
- [ ] Revoke GCP IAM `owner` permissions for `ondevice-platform-poc` project (Parker Cho held owner; Jason Park was admin).

### Section 13 — Harbor

- [ ] Remove OPaaS Docker image repositories from `harbor.dev.hpcnt.com`. Repos: `opaas-backend`, `opaas-frontend` images.

### Section 14 — HPCNT Artifact Registry (HAR)

- [ ] Verify that no active consumers reference `hpcnt-artifact-registry.svc.hpcnt.com/npm/npm-hpcnt/` for `opaas-frontend` packages.
- [ ] Remove opaas-frontend npm packages from HAR once confirmed no consumers remain.

### Section 15 — AWS account deprecation

- [ ] Confirm with HPCNT devops that no services other than OPaaS use the `mg-ai` AWS account.
- [ ] Deprecate (close) AWS `mg-ai` account after all OPaaS S3 and RDS resources are deleted.

### Section 16 — Git repository archival

- [ ] Archive `hyperconnect/opaas-backend` on `git.dev.hpcnt.com`.
- [ ] Archive `hyperconnect/opaas-frontend` on `git.dev.hpcnt.com`.
- [ ] Archive `hyperconnect/opaas-sdk-android` on `git.dev.hpcnt.com`.
- [ ] Archive `hyperconnect/opaas-sdk-ios` on `git.dev.hpcnt.com`.
- [ ] Archive `hyperconnect/opaas-model-tools` on `git.dev.hpcnt.com`.
- [ ] Archive or transfer `matchgroup-ai/opaas-sdk-android` on GitHub.
- [ ] Archive or transfer `matchgroup-ai/opaas-sdk-ios` on GitHub.

---

## Key Findings

### 1. Traffic is 0 and Hinge has permanently disabled the feature

Hinge disabled Photo Finder on 3/31/25 (confirmed by Doug Galante in #mgai_hinge_photo_picker_dev on 5/7/25). The Nov 2025 SRE task document (Owen Lee) states: "no brands are using OPaaS currently." Mumu communicated intent to sunset HPCNT infra to Kevin Sun and Doug Galante on 12/22/25 in #mgai_hinge_photo_picker_dev.

The OPaaS SDK may still be compiled into the Hinge app binary (feature disabled by lever), but this does not prevent server-side shutdown. Hinge removing the SDK from a future release is desirable but not a hard dependency.

### 2. Three environments are running with zero traffic

Each environment (prod, beta, qa) has 3 k8s deployments, 1 Aurora MySQL instance, and 2 S3 buckets. All are running on HPCNT-managed infrastructure and incurring cost with no active traffic.

- Prod and Beta share the cluster `kube-prod-uw2-ai-platform-hpcnt-com` (different namespaces: `opaas` and `beta-opaas`).
- QA runs on `kube-dev-uw2-ai-platform-hpcnt-com`, namespace `opaas`.

### 3. Cloudflare access is blocked — HPCNT devops required

MG AI cannot access the HPCNT Cloudflare environment due to SSO restrictions. This was noted by Dan Kim (HPCNT Infra) in Owen Lee's SRE doc. The same applies to Route53 hosted zone `Z0961120PZRFZ6FMZ1JZ`. HPCNT devops must perform Section 4 steps.

The SRE doc also notes: "Since OPaaS is not currently in use, removing Cloudflare could be a good option. (Access through the ALB should still remain available with the same domain.)"

### 4. Event Gateway dependency is unresolved

OPaaS SDK routes usage logs through HPCNT Core Platform's Event Gateway (separate from the k8s backend). The Nov 2025 SRE doc identifies this as an open question: whether to decommission it, find an MG Core (CPTS) alternative, or use an open-source replacement. Since this is a sunset (not a migration), decommission is expected — but requires HPCNT Core Platform team confirmation.

### 5. Okta migration was partially complete

Beta OPaaS was migrated from HCE Okta to MatchGroup Okta on 3/26/25 (Parker Cho, #mgai_hinge_photo_picker_dev). Production migration was planned to follow, but its completion status is unconfirmed. Both tenants must be checked.

### 6. CyberLink offline license requires no action

Hinge's CyberLink offline license (bundle IDs: `co.hinge.app`, `co.hinge.mobile.ios`) is configured server-side on OPaaS. Since OPaaS is being shut down, the license does not need to be renewed. No SDK update is required on Hinge's side to remove the license.

### 7. Kafka audit topics were never completed

The Notion API Server page records "Audit 연동 필요" (integration pending) next to Kafka Topics in all three environment tables. No Kafka topics for OPaaS audit appear to have been created. This item requires no action during shutdown.

---

## Evidence / Component Inventory

### K8s Deployments

| Deployment | Namespace | Cluster | Environment |
|---|---|---|---|
| opaas-admin-api-prod | opaas | kube-prod-uw2-ai-platform-hpcnt-com | prod |
| opaas-config-api-prod | opaas | kube-prod-uw2-ai-platform-hpcnt-com | prod |
| opaas-frontend-prod | opaas | kube-prod-uw2-ai-platform-hpcnt-com | prod |
| opaas-admin-api-prod | beta-opaas | kube-prod-uw2-ai-platform-hpcnt-com | beta |
| opaas-config-api-prod | beta-opaas | kube-prod-uw2-ai-platform-hpcnt-com | beta |
| opaas-frontend-prod | beta-opaas | kube-prod-uw2-ai-platform-hpcnt-com | beta |
| opaas-admin-api-prod | opaas | kube-dev-uw2-ai-platform-hpcnt-com | qa |
| opaas-config-api-prod | opaas | kube-dev-uw2-ai-platform-hpcnt-com | qa |
| opaas-frontend-prod | opaas | kube-dev-uw2-ai-platform-hpcnt-com | qa |

### Aurora MySQL Instances (AWS RDS, MySQL 8.0)

| Environment | Endpoint | DB |
|---|---|---|
| Prod | ai-platform-production-opaas.cluster-cvaecmfytoiu.us-west-2.rds.amazonaws.com:3306 | opaas |
| Beta | ai-platform-beta-opaas.cluster-cvaecmfytoiu.us-west-2.rds.amazonaws.com:3306 | opaas |
| QA | ai-platform-qa-opaas.cluster-cx42umccgb7k.us-west-2.rds.amazonaws.com:3306 | opaas |

### S3 Buckets (us-west-2)

| Bucket Name | Environment | Content |
|---|---|---|
| ai-platform-production-opaas-sdk-main-origin | prod | SDK release files |
| ai-platform-production-opaas-model-main-origin | prod | Encrypted ML model files (.tflite.enc) |
| ai-platform-beta-opaas-sdk-main-origin | beta | SDK release files |
| ai-platform-beta-opaas-model-main-origin | beta | Encrypted ML model files |
| ai-platform-qa-opaas-sdk-main-origin | qa | SDK release files |
| ai-platform-qa-opaas-model-main-origin | qa | Encrypted ML model files |

### API Endpoints

| Component | QA | Beta | Prod |
|---|---|---|---|
| Admin API | https://opaas-admin-api.dev.kube-uw2.aip.hpcnt.com | https://opaas-admin-api.beta.uw2.aip.hpcnt.com | https://opaas-admin-api.prod.kube-uw2.aip.hpcnt.com |
| Config API | https://opaas-config-api.dev.kube-uw2.aip.hpcnt.com | https://opaas-config-api.beta.uw2.aip.hpcnt.com | https://opaas-config-api.prod.kube-uw2.aip.hpcnt.com |
| Console | https://console.qa.opaas.hce.io | https://console.beta.opaas.hce.io | https://console.opaas.hce.io |

### Vault Secret Paths

| Secret Path | Vault Type | Environment | Service |
|---|---|---|---|
| kv-kube/kube-prod-uw2-ai-platform-hpcnt-com/opaas/opaas-admin-api-prod-secret | kv-kube | prod | admin-api (pod startup inject) |
| kv-kube/kube-prod-uw2-ai-platform-hpcnt-com/opaas/opaas-config-api-prod-secret | kv-kube | prod | config-api (pod startup inject) |
| kv-kube/kube-dev-uw2-ai-platform-hpcnt-com/opaas/opaas-admin-api-qa-secret | kv-kube | qa/beta | admin-api (pod startup inject) |
| kv-kube/kube-dev-uw2-ai-platform-hpcnt-com/opaas/opaas-config-api-qa-secret | kv-kube | qa/beta | config-api (pod startup inject) |
| kv-build/opaas-frontend/opaas-frontend/main/qa | kv-build | qa | frontend (Docker build-time inject) |
| kv-build/opaas-frontend/opaas-frontend/main/beta | kv-build | beta | frontend (Docker build-time inject) |
| kv-build/opaas-frontend/opaas-frontend/main/prod | kv-build | prod | frontend (Docker build-time inject) |

### Grafana Dashboards (base: https://grafana.kube-prod-an1.hpcnt.com)

| Dashboard ID | Environment | Metric |
|---|---|---|
| fdigombes34zkd | prod | admin-api system |
| fdigombes34zka | prod | config-api system |
| e36b46bf-f491-4606-a48b-9b04fc579385 | beta | admin-api system |
| cduz371jqo3y8f | beta | config-api system |
| edk4h5fx1v85cc | beta | config-api application |
| bdojbrnja3ke8c | qa | config-api system |

### Git Repositories

| Repo | Host | Purpose |
|---|---|---|
| hyperconnect/opaas-backend | git.dev.hpcnt.com | Admin API + Config API server |
| hyperconnect/opaas-frontend | git.dev.hpcnt.com | OPaaS Console (NextJS 13) |
| hyperconnect/opaas-sdk-android | git.dev.hpcnt.com | Android SDK |
| hyperconnect/opaas-sdk-ios | git.dev.hpcnt.com | iOS SDK |
| hyperconnect/opaas-model-tools | git.dev.hpcnt.com | Model conversion tooling (TFLite/MLModel) |
| hyperconnect/deploy-center | git.dev.hpcnt.com | Spinnaker deploy config (prod/main, beta/main, qa/main) |
| matchgroup-ai/opaas-sdk-android | github.com | Android SDK (external mirror) |
| matchgroup-ai/opaas-sdk-ios | github.com | iOS SDK (external mirror) |

---

## Reference (Appendix)

### Key contacts

| Person | Role | OPaaS involvement |
|---|---|---|
| Parker Cho | MLSE (MG AI) | Primary OPaaS V2 developer; Vault access, k8s deploys, Okta migration, CyberLink config |
| Joel Ham | MLSE (MG AI) | OPaaS SDK e2e test scripts; GCP/Firebase config (ondevice-platform-poc) |
| Mario Yoon | Engineer (HPCNT) | OPaaS server-side config, CyberLink license management for Hinge |
| Owen Lee | MLSE/TL (MG AI) | SRE owner; HPCNT infra migration planning |
| Jason Park | SRE/MLSE (MG AI) | GCP project admin (mgai-dev, ondevice-platform-poc) |
| Dan Kim | HPCNT Infra | Cloudflare and Route53 access (external dependency) |
| Kevin Sun | Hinge mobile lead | Photo Finder client-side; confirm permanent feature disable |
| Doug Galante | Hinge EM | Confirmed 3/31/25 feature disable in Slack |
| Niko Diakogiannis | Hinge iOS | OPaaS SDK integration |

### Cloudflare cache rules

- Zone ID: `59b61dfdaf996df7c8d763a0b7babd05` (domain: hce.io)
- Config API rule ID: `ed149e9e21b1485abd10964cfee76df1` (1 min TTL, applies to prod/beta/qa)
- Model Download rule ID: `51213d061f8d4d729f969b38c23b183c` (1 day TTL, applies to prod/beta/qa)

### Route53

- Hosted zone ID: `Z0961120PZRFZ6FMZ1JZ` (opaas-admin DNS records)

### Sentry projects

- Prod: project ID 630, URL `https://sentry.svc.hpcnt.com/organizations/hpcnt/projects/opaas-backend/`, environment filter `Production`
- Dev: project ID 464, URL `https://sentry-dev.svc.hpcnt.com/organizations/hpcnt/projects/opaas-backend/`

### Databricks

- Workspace: `data-infra-mg-ai-uw2` at https://dbc-d7b34265-86cb.cloud.databricks.com
- Metrics table: `opaas_prod.b__kafka.backend_platform_metric_in_house_app_opaas__view`

### Hinge SLA dashboards (for traffic verification)

- Hinge Dashboard: https://dbc-d03ede40-ab7e.cloud.databricks.com/dashboardsv3/01ef39c73ef014e29a928327cd9713e7/published?o=8844453302563067
- ASL Protoswipe Dashboard: https://dbc-d03ede40-ab7e.cloud.databricks.com/dashboardsv3/01ef76eb4c2e1630aaf9dd117746907c/published?o=8844453302563067

### GCP / Firebase PoC

- GCP project: `ondevice-platform-poc`
- Firebase console: https://console.firebase.google.com/project/ondevice-platform-poc/overview
- Cloud Run service: `collectfeedback` in `us-central1`
- GCP project (newer): `mgai-dev`
- Firebase console (newer): https://console.firebase.google.com/u/0/project/mgai-dev/overview

### OPaaS SDK ProjectKey note

Each OPaaS project (e.g., Hinge's photo selection project) holds a static `ProjectKey` used by the SDK to authenticate against config-api. These keys are stored in the Aurora MySQL DB (table `project_user_permission` and project settings). Deleting the database in Section 6 covers these keys; no separate API key revocation step is needed.

### Items not verified

- Whether Prod Okta migration (HCE → MatchGroup) was completed after 3/26/25. Beta was confirmed migrated; Prod status unknown.
- Whether ASL independently confirmed 0 traffic, or only inferred from Hinge's experiment disable.
- Whether any Kafka audit topics were ever created (Notion shows "Audit 연동 필요" — pending — suggesting they were not).
- Whether the `mg-ai` AWS account contains resources beyond OPaaS (S3, RDS). Verify before account closure.
