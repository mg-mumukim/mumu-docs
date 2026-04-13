# PART 1: Main Assessment - General Project Information
## 1.1 Who is the project manager or project lead?
This is the main point of contact who can best answer questions about this project, or identify team members to help answer questions.

Response:
Product manager
Software engineer

## 1.2 What is the Purpose of the Project?
Please respond to each of the following questions:
1. What is the purpose of this project?
2. Why is the project necessary?
3. Why does this project specifically require Personal Information to achieve the purpose?

NOTE: Please attach/link product specs, briefs, figma files and other relevant documentation to this assessment.

Response:
1. The purpose of this project is to utilize the LLM’s capability to understand Tinder profiles  across different elements such as photos and bios, and propose proper coaching, translating that understanding into actionable, data-backed coaching that can improve dating outcomes such as higher swipe right ratio.
- In the demo, the project will focus on using causal analysis and domain expertise to iteratively refine how the LLM can reliably interpret profile signals, distinguish between strong vs. weak first impressions, and provide evidence-backed coaching.
- This demo approach is also intentionally scoped as a foundation to:
  - Leverage the LLM to reason about profiles at a granular, interpretable level.
  - Leverage the ML models to extract domain concepts (e.g., has curly hair, outdoor) that contributes to dating outcomes in the vast user swipe logs.
  - Refine the AI-driven coaching experience in fast iterations.
  - Generate concrete learnings that inform Tinder’s broader profile-related initiatives without disrupting existing roadmaps and resource planning.

2. This project is necessary because while profile quality is the single most frequent, measurable, and leverageable determinant of downstream dating outcomes, users today receive limited or static guidance on how to improve their profiles. Specifically:
- Profile photos and presentations drive first impressions, but users lack personalized, evidence-backed feedback on what is hurting them. a level of dynamic and seamless personalization that can only be achieved through LLM generation.
- Existing heuristics (e.g. “add a social photo”) fail to account for context, segment differences, or interactions between different profile elements. Research on causal relationships between contents of profile and dating success is crucial.
- Before we rely on the LLM to influence higher-stakes moments (e.g. conversation), we must first demonstrate its reliable execution at the profile level.

3. This project requires Personal Information for two core reasons.
- First, evaluating and proposing improvements for an individual user’s profile is inherently personal, contextual, and user-specific.
- Second, to provide truly evidence-backed coaching, we must build statistical insights on the specific contexts and profile configurations where users have successfully achieved positive outcomes.

## 1.3 What is the duration of the project?
Response:
6-24 months

# PART 2: Main Assessment - Description of the Data Processing
## 2.1 Which category of "processing activity" does this project belong to?
A "processing activity" is a standard activity under which most projects can be categorized (e.g., operate the service, payment and transactions, Customer care, marketing / advertising…).

Response:
Analytics and Product Improvements | Tinder

## 2.2 How will Personal Information be processed?
After making your selection(s), please also provide, within the free text field below, an end-to-end description of how data will be processed as part of the project (e.g., where and how data will initially be collected, where it will be stored, how and by whom it will be used and how the data will be deleted...)

Response:
[ ] Collecting Personal Information
[ ] Storing Personal Information
[x] Using Personal Information
[ ] Deleting Personal Information
[ ] Sharing Personal Information
{{ to-do }}
{{ comments from legal team: the goal is to provide a step-by-step description of how data is used for the project, e.g. collecting from existing sources, sharing with LLM, insights generated, insights provided to users, etc. }}

## 2.6 Will any Personal Information processed as part of the project be transferred or accessible to a vendor / partner?
Identify any vendor(s), partner(s), or other third parties (including Public Authorities / Government Bodies / Law Enforcement) that support(s) this activity by processing Personal Information on behalf of Match Group or one of the MG Brands.

Response:
Information analysis or processing of the available user data is being done with GPT or Gemini.

## 2.7 What team(s) will have access to and use the Personal Information as part of the project?
Select the team(s) from the list below who require access to the Personal Information as part of the project.

After making your selection(s), please justify in the response field below:
why the selected team(s) have a need to access this information; and
what technical and/or organizational measures are put in place to ensure that only the selected team(s) will access the data.

Response:
[x] Engineering / Tech
[x] Product
The MGAI team ML Engineers, Software Engineers, Product Managers will be mainly interacting with the data. Some Engineers and Product members of the Tinder team will have access to the data via demo.

## 2.8 How long will the data be retained before it is anonymized or deleted?
Please explain how long and for what purpose each category of data will be retained as part of this project.

Please also describe how the data will be disposed of or anonymized at the end of the retention term e.g. automated processes, staff training, data audits.

For an explanation of permitted retention terms per category of data, please refer to Match Group's Data Lifecycle Policy accessible on Workday.

Response:
{{ TODO }}

## 2.9 What security measures will apply to the project?
Based on consultations with the Security team (or otherwise), please identify the security controls that have been / will be implemented.

Response:
This project will exclusively operate within tools and environments that have already been approved by the Match Group’s security review (e.g., GPT, Gemini, Databricks), No additional security controls are currently anticipated beyond existing safeguards. That said, we welcome guidance or recommendations for project-specific protections.
