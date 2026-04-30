<!-- Sources: note/opaas/2026-04-22-opaas-shutdown-actionable-guide-report-plan.md -->

# Handoff: OPaaS Sunset (v2 — Actionable Guide)

## Goal

For each checklist item in the OPaaS shutdown guide v1, provide concrete steps — interface, URL, permissions, commands — at Hyperconnect. Produces shutdown guide v2.

## Plan

**Question:** For each checklist item in the v1 shutdown guide, what are the concrete execution steps at Hyperconnect (interface, URL, required permissions, exact commands)?

**Sources:**
- Glean (chat + search) — primary source for how-to questions per infrastructure type: k8s/kubectl (kubeconfig, VPN, context), AWS console (RDS Aurora, S3), Vault UI/CLI (vault.kube-prod-an1.hpcnt.com), Grafana (grafana.kube-prod-an1.hpcnt.com), GitHub Enterprise (git.dev.hpcnt.com) repo archival, Harbor (harbor.dev.hpcnt.com) image cleanup, Okta app removal (HCE vs MatchGroup), GCP/Firebase cleanup, Databricks workspace, Sentry project deletion, Slack channel archival
- Notion — runbooks, onboarding docs, infra guides describing HPCNT tooling access patterns
- Slack (via Glean) — past discussions about infra access, permissions, VPN, kubeconfig setup

**Scope:**
- In scope: Concrete action items per checklist item — interface, URL, command, permission request process, and fallback if not found
- Out of scope: Re-investigation of component inventory (done in v1); changes to shutdown order or component list

**Type:** `-shutdown-guide`

**Status:** completed
