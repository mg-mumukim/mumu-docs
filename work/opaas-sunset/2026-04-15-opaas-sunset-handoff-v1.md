<!-- Sources: note/opaas/2026-04-15-opaas-sunset-report-plan.md -->

# Handoff: OPaaS Sunset

## Goal

Identify all running infrastructure components for OPaaS sunset — k8s workloads, DBs, Kafka topics, automation pipelines, API keys — with per-environment identifiers and correct shutdown order. Output: shutdown guide.

## Plan

**Question:** What components are currently running for OPaaS (On-device Platform as a Service), what are their identifiers per environment (prod/dev/qa), and what is the correct shutdown sequence?

**Sources:**
- Notion — OPaaS, Top Photo, Photo Ordering, Photo Selector, Photo Selection, Photo Finder, Photo Picker project pages, architecture docs, infra guides
- Glean — "OPaaS", "on-device platform", "top photo", "photo ordering", "photo selector" keywords
- Google Docs — design docs not in Notion, Hinge collaboration docs (via Glean)
- Jira (hyperconnect-atlassian) — OPaaS/Photo epics and tickets for infra component identifiers; key members: Parker, Joel
- Slack — engineering channels for OPaaS/photo service discussions, ops issues, shutdown discussions; key contact: Mario
- GitHub (matchgroup-ai) — deployed component names, k8s manifests, Kafka topic names from repo code

**Scope:**
- In scope: Infrastructure component list with per-environment identifiers, service dependency map, shutdown order and checklist
- Out of scope: ML model performance, business strategy, experiment results, service redesign proposals

**Type:** `-shutdown-guide`

**Status:** completed
