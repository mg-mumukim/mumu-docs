# OPaaS Sunset: Shutdown Guide

## Sources

- work/opaas-sunset/2026-04-15-opaas-sunset-shutdown-guide-v1.md (base document)
- Glean chat: Grafana access at HPCNT (CPTS VPN, Okta SSO, MG Central Grafana alternative)
- Glean chat: Kubernetes cluster access (VPN, kubeconfig, kubectl)
- Glean chat: GitHub Enterprise (git.dev.hpcnt.com) repo archival via DevOps Bot Web UI
- Glean chat: AWS console access via IAM Identity Center, awscreds 4.0.0
- Glean chat: Vault UI/CLI at vault.kube-prod-an1.hpcnt.com, token auth
- Glean chat: Okta app removal — MG Core Platform Tech Okta Groups/Permissions sheet
- Glean chat: Harbor at harbor.dev.hpcnt.com, robot account auth
- Slack #mgai-opaas, #mg-shared-vpn, #dev_devops_qna, #mg-core-infra-ai, #github-cloud-support, #mg_itsupport_apac, #devops-innovation-api
- Google Sheets: "MG CPTS VPN (MGCore VPN) Site List" by Owen Lee
- Google Sheets: "MG Core Platform Tech - Okta Groups/Permissions" by Sammie Cho
- Google Doc: "MG Shared VPN Proposal for MG AI Team" by Owen Lee

## Changes from v1

- Added concrete action items (interface, URL, command, who to contact) under every checklist item
- Added "Prerequisites" section covering VPN and AWS access setup needed before starting
- No changes to component inventory, shutdown order, or evidence tables (v1 is authoritative)

---

## Prerequisites

Complete these before starting the shutdown checklist.

### P1. VPN access

OPaaS infrastructure runs on HPCNT clusters. All HPCNT web consoles (Grafana, Vault, Harbor, Kibana, Sentry, git.dev.hpcnt.com) require VPN connectivity.

- [ ] Confirm you have CPTS VPN (MG Shared VPN) access.
  - **Action**: Open the "MG CPTS VPN (MGCore VPN) Site List" sheet by Owen Lee and verify the Grafana/Vault endpoints are listed.
  - **If not configured**: Ask Owen Lee or post in #mg-shared-vpn for VPN client setup instructions and credentials.
  - **Test**: Connect to VPN and open https://grafana.kube-prod-an1.hpcnt.com in a browser. If you see an Okta login page, VPN is working.

### P2. Hyperconnect Okta account

HPCNT web consoles authenticate via Hyperconnect Okta (not MatchGroup Okta).

- [ ] Confirm you can log in to Hyperconnect Okta.
  - **Action**: While on CPTS VPN, navigate to https://grafana.kube-prod-an1.hpcnt.com and attempt SSO login.
  - **If you lack an account**: Post in #mg_itsupport_apac requesting a Hyperconnect Okta account.

### P3. AWS IAM Identity Center access

AWS resources (Aurora RDS, S3) are in the `mg-ai` AWS account.

- [ ] Confirm you can access the mg-ai AWS account via IAM Identity Center.
  - **Action**: Click the "AWS IAM Identity Center" Okta tile. On the landing page, select the `mg-ai` account and your role.
  - **CLI access**: Install `awscreds 4.0.0` (IAM Identity Center). Run `awscreds` to authenticate. Use `--profile` for CLI operations.
  - **If your role is missing**: Post in #tinder-aws-iam-identity-migration. Roles unused for 3+ years were not migrated.

### P4. Kubernetes access

- [ ] Obtain kubeconfig for the two clusters: `kube-prod-uw2-ai-platform-hpcnt-com` and `kube-dev-uw2-ai-platform-hpcnt-com`.
  - **Action**: Ask Owen Lee or HPCNT DevOps team for the kubeconfig files. There is no self-service portal; kubeconfigs are distributed by SRE/DevOps.
  - **Test**: `kubectl --context=kube-prod-uw2-ai-platform-hpcnt-com get namespaces | grep opaas`

---

## Ordered Shutdown Checklist

### Section 1 — Confirm zero traffic and coordinate with Hinge

- [ ] Verify zero traffic in Grafana dashboards for all 3 environments (prod, beta, qa).
  - **Interface**: Web browser (CPTS VPN required)
  - **Action**:
    1. Connect to CPTS VPN.
    2. Open https://grafana.kube-prod-an1.hpcnt.com and log in via Okta SSO.
    3. Navigate to each dashboard by appending `/d/<dashboard-id>` to the base URL:
       - Prod admin-api: `/d/fdigombes34zkd`
       - Prod config-api: `/d/fdigombes34zka`
       - Beta admin-api: `/d/e36b46bf-f491-4606-a48b-9b04fc579385`
       - Beta config-api: `/d/cduz371jqo3y8f`
       - Beta config-api app: `/d/edk4h5fx1v85cc`
       - QA config-api: `/d/bdojbrnja3ke8c`
    4. Set time range to "Last 30 days". Confirm all request rate panels show 0.
  - **Alternative**: If HPCNT Grafana access fails, try MG Central Grafana at https://grafana.matchgroupcentral.net — HPCNT Prometheus metrics may be federated there. Search for dashboards containing "opaas".
  - **Also check**: Hinge SLA dashboard at https://dbc-d03ede40-ab7e.cloud.databricks.com/dashboardsv3/01ef39c73ef014e29a928327cd9713e7/published?o=8844453302563067

- [ ] Confirm with Hinge (Kevin Sun, Doug Galante) that Photo Finder feature flag will remain off permanently and SDK will be removed in a future app release.
  - **Action**: Send message in #mgai_hinge_photo_picker_dev (Tinder Slack) tagging Kevin Sun and Doug Galante.
  - **Ask**: "Can you confirm the Photo Finder feature flag will stay off permanently, and whether the OPaaS SDK removal is planned for a future Hinge release?"

- [ ] Confirm with ASL (Art & Science Lab / Tinder) that no OPaaS SDK usage is active.
  - **Action**: Search #mgai-dev (Tinder Slack) for past ASL discussions. If no contact found, ask Owen Lee for the ASL point of contact.

### Section 2 — Disable automated deployments (CI/CD)

- [ ] Disable GitHub Actions in `hyperconnect/opaas-backend` on git.dev.hpcnt.com.
  - **Interface**: Web browser (CPTS VPN required)
  - **Action**:
    1. Navigate to https://git.dev.hpcnt.com/hyperconnect/opaas-backend/settings/actions
    2. Under "Actions permissions", select "Disable Actions" and save.
    3. Alternatively, delete or rename `.github/workflows/ci.yml` and `.github/workflows/cd.yml` via a commit.

- [ ] Disable GitHub Actions in `hyperconnect/opaas-frontend` on git.dev.hpcnt.com.
  - **Action**: Same as above at https://git.dev.hpcnt.com/hyperconnect/opaas-frontend/settings/actions

- [ ] Remove OPaaS entries from deploy-center.
  - **Interface**: git (push commits)
  - **Action**:
    1. Clone `git.dev.hpcnt.com/hyperconnect/deploy-center`.
    2. In branches `prod/main`, `beta/main`, `qa/main`: find and remove OPaaS deployment entries (search for "opaas" in the config files).
    3. Create a PR for each branch. Get approval and merge.
  - **If you lack push access**: Ask Owen Lee or HPCNT DevOps for write access to deploy-center.

### Section 3 — Scale down k8s deployments (QA -> Beta -> Prod)

All 3 deployments per namespace: `opaas-admin-api-prod`, `opaas-config-api-prod`, `opaas-frontend-prod`.

- [ ] **QA**: Scale to 0 in namespace `opaas`, cluster `kube-dev-uw2-ai-platform-hpcnt-com`.
  - **Interface**: Terminal (kubectl)
  - **Action**:
    ```
    kubectl --context=kube-dev-uw2-ai-platform-hpcnt-com -n opaas scale deployment opaas-admin-api-prod --replicas=0
    kubectl --context=kube-dev-uw2-ai-platform-hpcnt-com -n opaas scale deployment opaas-config-api-prod --replicas=0
    kubectl --context=kube-dev-uw2-ai-platform-hpcnt-com -n opaas scale deployment opaas-frontend-prod --replicas=0
    ```
  - **Verify**: `kubectl --context=kube-dev-uw2-ai-platform-hpcnt-com -n opaas get deployments` — all should show 0/0 READY.

- [ ] **Beta**: Scale to 0 in namespace `beta-opaas`, cluster `kube-prod-uw2-ai-platform-hpcnt-com`.
  - **Action**:
    ```
    kubectl --context=kube-prod-uw2-ai-platform-hpcnt-com -n beta-opaas scale deployment opaas-admin-api-prod --replicas=0
    kubectl --context=kube-prod-uw2-ai-platform-hpcnt-com -n beta-opaas scale deployment opaas-config-api-prod --replicas=0
    kubectl --context=kube-prod-uw2-ai-platform-hpcnt-com -n beta-opaas scale deployment opaas-frontend-prod --replicas=0
    ```

- [ ] **Prod**: Scale to 0 in namespace `opaas`, cluster `kube-prod-uw2-ai-platform-hpcnt-com`.
  - **Action**:
    ```
    kubectl --context=kube-prod-uw2-ai-platform-hpcnt-com -n opaas scale deployment opaas-admin-api-prod --replicas=0
    kubectl --context=kube-prod-uw2-ai-platform-hpcnt-com -n opaas scale deployment opaas-config-api-prod --replicas=0
    kubectl --context=kube-prod-uw2-ai-platform-hpcnt-com -n opaas scale deployment opaas-frontend-prod --replicas=0
    ```

### Section 4 — CDN and DNS (requires HPCNT DevOps)

MG AI cannot access HPCNT Cloudflare due to SSO restrictions. All items in this section must be performed by Dan Kim (HPCNT Infra).

- [ ] Remove Cloudflare CDN cache rule for Config API.
  - **Action**: Message Dan Kim in #mgai-opaas or DM. Provide:
    - Zone ID: `59b61dfdaf996df7c8d763a0b7babd05` (domain: hce.io)
    - Rule ID: `ed149e9e21b1485abd10964cfee76df1` (Config API, 1 min TTL)
    - Dashboard link: https://dash.cloudflare.com/59b61dfdaf996df7c8d763a0b7babd05/hce.io/caching/cache-rules

- [ ] Remove Cloudflare CDN cache rule for Model Download.
  - **Action**: Same request to Dan Kim. Rule ID: `51213d061f8d4d729f969b38c23b183c` (Model Download, 1 day TTL).

- [ ] Remove Route53 DNS records for `opaas-admin`.
  - **Action**: Same request to Dan Kim. Hosted zone ID: `Z0961120PZRFZ6FMZ1JZ`.

### Section 5 — Okta

Beta was migrated from HCE Okta to MatchGroup Okta on 3/26/25 (Parker Cho). Prod migration status is unconfirmed.

- [ ] Remove OPaaS application from HCE Okta (production console auth).
  - **Interface**: Okta admin console (requires Okta admin privileges)
  - **Action**:
    1. Open the "MG Core Platform Tech - Okta Groups/Permissions" Google Sheet (by Sammie Cho) to find who has HCE Okta admin access.
    2. If you do not have admin access, request removal via #mg_itsupport_apac, specifying: "Remove the OPaaS Console application from HCE (Hyperconnect) Okta. The service is being decommissioned."
    3. If you have admin access: navigate to the HCE Okta admin console > Applications > search "OPaaS" > Deactivate > Delete.
  - **Investigate first**: Confirm whether the prod Okta migration from HCE to MG was completed after 3/26/25. Ask Parker Cho in #mgai-opaas.

- [ ] Remove OPaaS application from MatchGroup Okta (beta console).
  - **Action**: Same process as above, but targeting the MatchGroup Okta tenant. Use the same Google Sheet to find MG Okta admins.

### Section 6 — Databases (backup first, then delete)

Snapshot all Aurora instances before deletion. DB spec: MySQL 8.0. All in us-west-2.

- [ ] Snapshot QA Aurora.
  - **Interface**: AWS Console (Web) or CLI
  - **Action (Console)**:
    1. Log in to AWS console via IAM Identity Center, select `mg-ai` account.
    2. Navigate to RDS > Databases > find `ai-platform-qa-opaas`.
    3. Select instance > Actions > Take snapshot. Name it `opaas-qa-final-snapshot-2026-MM-DD`.
  - **Action (CLI)**:
    ```
    aws rds create-db-cluster-snapshot \
      --db-cluster-identifier ai-platform-qa-opaas \
      --db-cluster-snapshot-identifier opaas-qa-final-snapshot-2026-04-22 \
      --region us-west-2
    ```
  - **If you lack RDS permissions**: Ask Jason Park or Sammie Cho for IAM role escalation, or post in #mg-core-infra-ops.

- [ ] Snapshot Beta Aurora.
  - **Action**: Same process for `ai-platform-beta-opaas`. Snapshot name: `opaas-beta-final-snapshot-2026-MM-DD`.

- [ ] Snapshot Prod Aurora.
  - **Action**: Same process for `ai-platform-production-opaas`. Snapshot name: `opaas-prod-final-snapshot-2026-MM-DD`.

- [ ] Delete QA Aurora instance.
  - **Action (Console)**: RDS > Databases > select `ai-platform-qa-opaas` > Actions > Delete. Uncheck "Create final snapshot" (already taken). Confirm deletion.
  - **Action (CLI)**: `aws rds delete-db-cluster --db-cluster-identifier ai-platform-qa-opaas --skip-final-snapshot --region us-west-2`
  - **Note**: For production infra deletion, an OPS ticket may be required. Check with HPCNT DevOps whether the `mg-ai` account follows the same OPS ticket process as the main Tinder accounts.

- [ ] Delete Beta Aurora instance.
  - **Action**: Same process for `ai-platform-beta-opaas`.

- [ ] Delete Prod Aurora instance.
  - **Action**: Same process for `ai-platform-production-opaas`. Exercise extra caution — confirm snapshot is complete before deletion.

### Section 7 — S3 Buckets (archive or delete)

6 buckets total, all in us-west-2. Consider archiving to Glacier before full deletion.

- [ ] Archive/delete each bucket.
  - **Interface**: AWS Console or CLI
  - **Action per bucket**:
    1. AWS Console > S3 > find bucket name.
    2. Review contents. If archival is desired, set lifecycle policy to transition to Glacier first.
    3. To delete: empty the bucket first (`aws s3 rm s3://<bucket-name> --recursive`), then delete the bucket (`aws s3 rb s3://<bucket-name>`).
  - **Buckets**: `ai-platform-production-opaas-sdk-main-origin`, `ai-platform-production-opaas-model-main-origin`, `ai-platform-beta-opaas-sdk-main-origin`, `ai-platform-beta-opaas-model-main-origin`, `ai-platform-qa-opaas-sdk-main-origin`, `ai-platform-qa-opaas-model-main-origin`
  - **Note**: For production S3 deletion, check if a PR in `terra-aws-coe` repo is required (Terraform-managed infrastructure). Ask Sammie Cho or post in #mg-core-infra-ai.

### Section 8 — Vault Secrets

Backend secrets are injected at pod startup (kv-kube). Frontend secrets are injected at Docker build time (kv-build). Delete after services are confirmed stopped.

- [ ] Delete all 7 Vault secret paths.
  - **Interface**: Web browser (CPTS VPN required)
  - **Action**:
    1. Connect to CPTS VPN.
    2. Open https://vault.kube-prod-an1.hpcnt.com/ui/ and authenticate with your Vault token.
    3. Navigate to each secret path and click "Delete" (or "Destroy" for permanent removal):
       - `kv-kube/kube-prod-uw2-ai-platform-hpcnt-com/opaas/opaas-admin-api-prod-secret`
       - `kv-kube/kube-prod-uw2-ai-platform-hpcnt-com/opaas/opaas-config-api-prod-secret`
       - `kv-kube/kube-dev-uw2-ai-platform-hpcnt-com/opaas/opaas-admin-api-qa-secret`
       - `kv-kube/kube-dev-uw2-ai-platform-hpcnt-com/opaas/opaas-config-api-qa-secret`
       - `kv-build/opaas-frontend/opaas-frontend/main/qa`
       - `kv-build/opaas-frontend/opaas-frontend/main/beta`
       - `kv-build/opaas-frontend/opaas-frontend/main/prod`
  - **If you get 403**: Your Vault token lacks delete permissions on these paths. Request a new token from HPCNT DevOps via #dev_devops_qna, specifying the exact paths and that you need delete permission.
  - **Note**: Vault at HPCNT may be migrating to AWS Secrets Manager. If the migration already affected these paths, check if they still exist in Vault or have moved.

### Section 9 — Event Gateway

OPaaS SDK sends usage logs to HPCNT Core Platform's Event Gateway. Traffic is already 0.

- [ ] Confirm with HPCNT Core Platform team whether the Event Gateway instance is OPaaS-exclusive or shared.
  - **Action**: Post in #mgai-opaas tagging the HPCNT Core Platform team contact (look up in Owen Lee's SRE doc). Ask: "Is the Event Gateway instance used by OPaaS shared with other services, or is it exclusive to OPaaS?"

- [ ] If exclusive: request decommission. If shared: request OPaaS consumer removal.
  - **Action**: Based on the answer, file a request with the HPCNT Core Platform team in the appropriate channel.

### Section 10 — Monitoring and alerting

- [ ] Archive or delete 6 Grafana dashboards.
  - **Interface**: Web browser (CPTS VPN required)
  - **Action**:
    1. Connect to CPTS VPN. Open https://grafana.kube-prod-an1.hpcnt.com.
    2. For each dashboard ID (listed in v1 Evidence section), navigate to `/d/<id>`, click the gear icon (Dashboard settings) > scroll to bottom > "Delete Dashboard".
    3. If you lack Grafana editor permissions, ask in #dev_devops_qna to be granted editor role on these dashboards.

- [ ] Archive or delete Kibana APM services.
  - **Interface**: Web browser (CPTS VPN required)
  - **Action**: Navigate to https://kibana-sso.kube-prod-an1.hpcnt.com > APM > Services. Find `platform-opaas-admin-api` and `platform-opaas-config-api`. Kibana APM services are typically auto-cleaned when no data flows, but verify they are no longer listed.
  - **If manual deletion is needed**: Kibana APM services are backed by Elasticsearch indices. Contact HPCNT SRE via #dev_devops_qna to request index cleanup for OPaaS APM data.

- [ ] Delete Sentry prod project.
  - **Interface**: Web browser (CPTS VPN required)
  - **Action**: Open https://sentry.svc.hpcnt.com/organizations/hpcnt/projects/opaas-backend/ > Settings > scroll to bottom > "Remove Project". Confirm deletion.
  - **If you lack admin access**: Ask in #dev_devops_qna for Sentry project admin on `opaas-backend`.

- [ ] Delete Sentry dev project.
  - **Action**: Same process at https://sentry-dev.svc.hpcnt.com/organizations/hpcnt/projects/opaas-backend/

- [ ] Archive Slack channel #opaas_oncall.
  - **Interface**: Slack (Hyperconnect workspace)
  - **Action**: Open #opaas_oncall > click channel name header > Settings > Archive channel. You need to be a channel admin or workspace admin.
  - **If you lack permission**: Ask in #mgai-opaas for someone with admin access to archive it.

### Section 11 — Databricks

- [ ] Archive table `opaas_prod.b__kafka.backend_platform_metric_in_house_app_opaas__view`.
  - **Interface**: Web browser
  - **Action**: Open https://dbc-d7b34265-86cb.cloud.databricks.com > navigate to the `data-infra-mg-ai-uw2` workspace > SQL editor > run `DROP TABLE IF EXISTS opaas_prod.b__kafka.backend_platform_metric_in_house_app_opaas__view;`
  - **If you lack workspace access**: Ask Jason Park or Owen Lee for Databricks workspace access.

- [ ] Evaluate whether `data-infra-mg-ai-uw2` workspace can be fully deprecated.
  - **Action**: In the workspace, check SQL editor > catalog browser for any non-OPaaS tables or notebooks. If empty, request workspace deletion from the Databricks admin.
  - **Investigate**: Ask Owen Lee — his SRE doc may have details on other workloads in this workspace.

### Section 12 — GCP / Firebase PoC resources

Contact Jason Park for GCP admin access.

- [ ] Delete Cloud Run service `collectfeedback` in `us-central1`, project `ondevice-platform-poc`.
  - **Interface**: Web browser
  - **Action**: Open https://console.cloud.google.com/run?project=ondevice-platform-poc > select `collectfeedback` > Delete.
  - **If you lack access**: Ask Jason Park for IAM Owner or Editor role on `ondevice-platform-poc`.

- [ ] Delete or verify Firebase project `ondevice-platform-poc`.
  - **Action**: Open https://console.firebase.google.com/project/ondevice-platform-poc/overview > Project settings > scroll to bottom > "Delete project".
  - **Investigate first**: Earlier Slack suggests it may have been abandoned in favor of `mgai-dev`. Confirm with Joel Ham.

- [ ] Verify Firebase project `mgai-dev` — delete OPaaS-related apps only.
  - **Action**: Open https://console.firebase.google.com/u/0/project/mgai-dev/overview > Project settings > "Your apps" section. Find `mgai.tailor.playground` iOS app. Delete it if not used by other workloads.
  - **Investigate**: Confirm with Jason Park whether `mgai-dev` is used by any other team.

- [ ] Revoke GCP IAM permissions for `ondevice-platform-poc`.
  - **Action**: GCP Console > IAM > select `ondevice-platform-poc` project > review members. Remove Owner role from Parker Cho and admin from Jason Park (after confirming project is deleted or abandoned).

### Section 13 — Harbor

- [ ] Remove OPaaS Docker image repositories from Harbor.
  - **Interface**: Web browser (CPTS VPN required)
  - **Action**:
    1. Connect to CPTS VPN. Open https://harbor.dev.hpcnt.com.
    2. Log in. If prompted for credentials, use the robot account `robot$harbor-action-bot` (password stored in CI/CD secrets — ask HPCNT DevOps if needed). Alternatively, your HPCNT Okta credentials may work if SSO is configured.
    3. Navigate to the project containing `opaas-backend` and `opaas-frontend` images.
    4. Select each repository > Actions > Delete.
  - **If repos are not visible**: Harbor visibility is per-project. Ask in #dev_devops_qna for access to the OPaaS-related Harbor project.

### Section 14 — HPCNT Artifact Registry (HAR)

- [ ] Verify no active consumers reference OPaaS packages.
  - **Action**: Search the `hyperconnect/opaas-frontend` repo on git.dev.hpcnt.com for `hpcnt-artifact-registry.svc.hpcnt.com/npm/npm-hpcnt/` references. Also search #mgai-opaas for any mention of HAR package consumers.
  - **If no consumers found**: Proceed to removal.

- [ ] Remove opaas-frontend npm packages from HAR.
  - **Action**: Navigate to `https://hpcnt-artifact-registry.svc.hpcnt.com` (CPTS VPN required). Find and delete the opaas-frontend npm packages.
  - **If you lack access**: Ask HPCNT DevOps via #dev_devops_qna for HAR admin access to delete the packages.

### Section 15 — AWS account deprecation

- [ ] Confirm no services other than OPaaS use the `mg-ai` AWS account.
  - **Action**: Log in to the `mg-ai` AWS account via IAM Identity Center. Check:
    1. RDS > any remaining instances?
    2. S3 > any remaining buckets?
    3. EC2 > any running instances?
    4. Lambda > any functions?
    5. CloudFormation > any stacks?
  - **Also ask**: Post in #mg-core-infra-ai: "Can you confirm whether the mg-ai AWS account contains any resources other than OPaaS?"

- [ ] Deprecate (close) the `mg-ai` AWS account.
  - **Action**: AWS account closure requires MG Infra team involvement. Post in #mg-core-infra-ops requesting account closure for `mg-ai`, with confirmation that all resources have been deleted.
  - **Note**: Account closure is irreversible after a 90-day grace period. Ensure all snapshots are retained in a separate account if needed.

### Section 16 — Git repository archival

- [ ] Archive 5 repos on git.dev.hpcnt.com.
  - **Interface**: Web browser (CPTS VPN required)
  - **Action per repo**:
    1. Navigate to the DevOps Bot Web UI: `https://devops-bot.svc.hpcnt.com/ui/git/repos/git.dev.hpcnt.com/hyperconnect/<repo-name>`
    2. Find the archive option and apply.
    3. If the archive action is restricted, tag HPCNT DevOps (Gom.S / Kyusung Seo) in #dev_devops_qna and request archival.
  - **Repos**: `opaas-backend`, `opaas-frontend`, `opaas-sdk-android`, `opaas-sdk-ios`, `opaas-model-tools`

- [ ] Archive or transfer 2 repos on github.com.
  - **Interface**: Web browser (GitHub.com)
  - **Action per repo**:
    1. Navigate to https://github.com/matchgroup-ai/<repo-name>/settings
    2. Scroll to "Danger Zone" > "Archive this repository".
    3. Requires repo admin permission. If you lack it, ask Owen Lee for admin access to the matchgroup-ai org.
  - **Repos**: `opaas-sdk-android`, `opaas-sdk-ios`

---

## Key Findings

Unchanged from v1 — see the v1 document for detailed findings on traffic status, environment topology, Cloudflare access restrictions, Event Gateway dependency, Okta migration status, CyberLink license, and Kafka audit topics.

---

## Evidence / Component Inventory

Unchanged from v1 — see the v1 document for complete tables of k8s deployments, Aurora instances, S3 buckets, API endpoints, Vault paths, Grafana dashboards, git repositories, contacts, Cloudflare rules, Route53, Sentry projects, Databricks, Hinge SLA dashboards, GCP/Firebase resources, and unverified items.
