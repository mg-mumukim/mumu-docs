# OPaaS Shutdown Actionable Guide — Report Plan

## Question
For each checklist item in the OPaaS shutdown guide v1, what are the concrete steps — which interface (web console, terminal, API), which URL, what permissions, and what commands — to execute that item at Hyperconnect?

## Sources to check
- **Glean (chat + search)** — Primary source. Ask how-to questions for each infrastructure type at Hyperconnect/HPCNT:
  - k8s cluster access and `kubectl` usage (kubeconfig, VPN, context switching)
  - AWS console access for RDS (Aurora snapshots, deletion) and S3 (bucket lifecycle, deletion)
  - Vault UI and CLI usage (`vault.kube-prod-an1.hpcnt.com`)
  - Grafana dashboard access (`grafana.kube-prod-an1.hpcnt.com`)
  - GitHub Enterprise (`git.dev.hpcnt.com`) repo archival workflow
  - Harbor (`harbor.dev.hpcnt.com`) image cleanup
  - Okta app removal process (HCE Okta vs MatchGroup Okta)
  - GCP project deletion and Firebase cleanup
  - Databricks workspace/table management
  - Sentry project deletion
  - Slack channel archival
- **Notion** — Search for runbooks, onboarding docs, infra guides that describe HPCNT tooling access patterns
- **Slack (via Glean)** — Past discussions about infra access, permissions, VPN, kubeconfig setup

## Scope
- **In**: Concrete action items per checklist item — interface, URL, command, permission request process, and "investigate this if not found" fallback
- **Out**: Re-investigation of component inventory (already done in v1). No changes to shutdown order or component list.

## Output
`-shutdown-guide` (v2 of existing guide, augmented with action items)
