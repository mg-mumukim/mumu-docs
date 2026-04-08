# Privacy by Design - Section 2.2 Data Flow Description

## Goal
Complete section 2.2 of the AURA Demo Privacy by Design assessment form (`00_note/2026-04-03-privacy-by-design-aura-demo-wip.md`).

Section 2.2 asks "How will Personal Information be processed?" with a free-text field requesting an end-to-end description of the data flow. The legal team explicitly requested a step-by-step description.

## Source Material
- `00_note/2026-04-03-privacy-by-design-system-context.md` -- system description and legal risk categories
- `00_note/2026-03-18-legal-privacy-touch-base-meeting-note.md` -- legal team action items
- `00_note/2026-04-08-aura-demo-prompt.md` -- demo prompt guardrails and output format
- `00_note/2026-04-08-aura-research-agents.md` -- research agent workflow
- `00_note/2026-04-08-dating-concept-list.csv` -- domain concept list with coaching eligibility

## Legal Team Requirements
From the legal team's comment:
> the goal is to provide a step-by-step description of how data is used for the project,
> - collecting from existing sources
> - sharing with LLM
> - insights generated
> - insights provided to users, etc.

From the meeting note action items:
> - List of data that is being used in the Knowledge Base + what kind of insights we are looking for from the Knowledge Base + if the data is being moved outside of the KB, where it goes
> - Double check if LLMs does not extract biometric data from user

## Structure
An 8-step data flow description:

1. **Data Collection** -- Data categories (demographics, user-selected, UGC, behavior, technical flags), geographic scope, exclusions, data management, biometric data note with gray-zone guardrails
2. **Research** -- 7-step process (cohort segmentation -> hypothesis -> querying -> LLM classification -> statistical analysis -> report -> human review); KB outputs are aggregate-level, not Personal Information
3. **Custom Model Training** -- 3 objectives (concept discovery, concept prediction, causal inference with CATE); demographics as conditioning variables, behavior data as outcome signal
4. **Demo** -- Custom model concept extraction -> LLM API (Gemini) coaching generation -> user output; sensitive data handling for bio text and unblurred photos; concept lifecycle (extraction, curation, coaching eligibility with coachable/non-coachable classification)
5. **Evaluation** -- Human and/or LLM-as-a-judge against 8 criteria; also serves as continuous monitoring for guardrail compliance (bio safety, photo identification, sensitive concept surfacing)
6. **Third-Party LLM Vendors** -- Claude and ChatGPT for Research/Evaluation; Gemini for Research/Demo/Evaluation
7. **Output and Access** -- KB and models (internal), coaching (end user), evaluation results (internal)
8. **Data Retention** -- 3-month removal cycle, DLP-governed, no indefinite retention

## Key Privacy Considerations Addressed
- User identifiers (uid, user number, name, email) excluded from all processing
- Geographic exclusions: EU/EEA/Switzerland/US-Illinois
- Biometric data not extracted; gray-zone risk mitigated via prompt guardrails
- Photos passed unblurred (required for expression/composition evaluation); no facial identification/verification performed
- Bio text may contain sensitive data but coachable topics are pre-fixed; compliance monitored via evaluation pipeline
- Domain concepts with legal extraction risk removed entirely; sensitive-but-extractable concepts marked non-coachable
- KB outputs are aggregate statistics, human-reviewed before export
- All concept lists human-confirmed before deployment
