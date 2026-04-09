# Plan: Privacy by Design - Section 2.8 (Data Retention and Deletion)

Target document: `00_note/2026-04-03-privacy-by-design-aura-demo-wip.md` section 2.8

---

## Goal

Section 2.8 asks: "How long will the data be retained before it is anonymized or deleted?"
Legal needs a clear picture of retention timelines, deletion mechanisms (automated vs manual), and current gaps per environment.

## Source material

1. `00_note/2026-04-03-privacy-by-design-system-context.md` -- Data lifecycle policy section (3 environments)
2. `00_note/2026-04-06-hyperpod-dlp-plan-mg-security.md` -- HyperPod DLP engineering details
3. `01_work/01_draft/2026-04-08-privacy-by-design-section-2-2.md` -- Step 8 (brief retention mention) and overall style reference

## Structure

Section 2.8 will be organized by environment, since each has different retention rules, deletion mechanisms, and maturity levels. For each environment:

| Aspect | What legal needs to see |
|--------|------------------------|
| What data is stored | Type and scope |
| Retention period | Concrete timeline |
| Deletion trigger | What initiates deletion (time-based, account event, manual) |
| Deletion mechanism | Automated pipeline vs manual process |
| Verification | How deletion is confirmed |
| Current status | What works now vs what is being built |

### Environments to cover

1. **HyperPod (Custom model training)** -- richest detail available from DLP plan
   - Structured data via Delta Sharing, unstructured via S3
   - 3-month retention, manual deletion transitioning to automated pipeline
   - Deactivated account flush: event-driven detection + 48h SLA
   - S3 inventory snapshot verification
   - Audit logging
   - Phase 1 (current) vs Phase 2 (target) distinction

2. **Local machines (Agentic research)** -- least controlled, flag as gap
   - Data analysis on engineer macbooks
   - No automated deletion pipeline
   - Relies on manual deletion discipline
   - Note: needs organizational/policy controls

3. **MG Core S3 (AURA demo and evaluation)** -- intermediate
   - Copied user profiles for demo UI and LLM input
   - 3-month retention aligned with MG DLP
   - Deletion policy to be confirmed

### Cross-cutting points

- Source data governed by MG DLP upstream
- Geographic scope exclusions (EU/EEA/Switzerland/US-Illinois) reduce retention risk surface
- No project-specific data retained indefinitely
- Knowledge Base and model weights do not contain Personal Information and follow separate retention

## Style

- Match the tone and structure of section 2.2 draft (structured, table-assisted, direct)
- Shorter than 2.2 -- this is a focused retention section, not a full data flow description
- Clearly distinguish current state vs planned improvements for legal transparency
