# Legal
## Risk category identified
### 1. External LLM Integration and Data Leakage Risks
This category covers the privacy and security vulnerabilities that arise when transmitting user information to third-party AI models and processors.
Key Risks: Sending raw profile photos and full, unredacted bio text/chat data to external LLM vendors (such as Google Gemini and OpenAI) significantly increases the risk of a security breach or data leakage. The lack of control over how third-party APIs process and handle this raw data creates critical exposure points.

### 2. Internal Data Security, Access Control, and Lifecycle Management
This category highlights internal infrastructure vulnerabilities related to how extracted user data is stored, accessed, and retained within the company's systems.
Key Risks: Extracted data is stored in local CSV files on disk without encryption at rest, and lacks data anonymization (e.g., user number is stored as-is). Furthermore, all extracted data is retained indefinitely without an automated deletion process, and access is poorly controlled by relying on a single shared Databricks Personal Access Token (PAT) for all access rather than scoped permissions.

### 3. Sensitive Inferences and AI Unpredictability (Hallucinations)
This category addresses the risks associated with the AI model's unpredictable outputs, biases, and its potential to deduce protected user information from UGC (bio and photos).
Key Risks: The LLM's outputs (tags or embeddings) may hallucinate, misrepresent user behavior, or encode bias, leading to inaccurate or counterproductive recommendations. More critically, the AI may incorrectly infer highly sensitive data categories (such as sexual orientation, health, political, or religious beliefs) from a user's profile. This creates severe PR and legal risks, especially regarding users' rights to access, correct, and delete data generated about them.

### 4. Section 230 Immunity and Profile Authenticity
This category involves legal threats to the platform's liability protections and the potential misuse of the feature by malicious users.
Key Risks: By actively providing feedback and assisting in the creation of user profiles, the platform could be viewed as a co-creator, which jeopardizes its Section 230 protection (a law that only shields platforms from liability for purely user-generated content). Additionally, bad actors and scammers could leverage this AI assistance to craft highly effective, deceptive profiles, which would ultimately diminish the overall authenticity of the platform.

### 5. Biometric Data Compliance and Consent Management
This category addresses the strict legal and regulatory vulnerabilities surrounding the collection, storage, and matching of users' facial and biometric data.
Key Risks: Implementing facial recognition models (such as 1:N identification or 1:1 verification) introduces severe legal exposure under stringent privacy laws like BIPA, CCPA, and GDPR. Processing facial geometries (FaceMaps) without obtaining explicit, bifurcated user consent for short-term versus long-term storage can lead to massive financial penalties. Furthermore, the lack of automated Data Lifecycle Policy (DLP) pipelines to immediately delete biometric data when a user revokes consent, deletes their account, or fails verification constitutes an unacceptable regulatory risk.

## Legal team comments for data processing system
the goal is to provide a step-by-step description of how data is used for the project,
- collecting from existing sources
- sharing with LLM
- insights generated
- insights provided to users, etc.

# System description
## 1. Tinder user data
### Contents
We use user information including
1. Demographics excluding user identifers (uid, user number, name, email address)
  - age
  - gender (male/female/non-binary)
  - target gender (male/female/non-binary)
  - country code (e.g., US)
  - city (e.g., New York)
  - language (e.g., English)

Demographic features are used to differentiate appropriate treatment for the user, because useful and acceptable advices are different along cohorts. For example, height shows different effect between different genders.

2. User selected contents
  - job titles, employers, school
  - lifestyle (e.g., drinking, smoking, workout, dietary preference, ...)
  - basics (e.g., zodiac, education, family plan, communication style, ...)
  - interests (e.g., travel, hiking, coffee, ...)
  - relationsip intent (e.g., long-term)

3. User generated contents
  - bio (text)
  - photos (image)

User selected and generated contents are used to generate personalized treatment for the user. For example, LLM decides to give a review "Add hiking photo" to the user because (1) the user has selected "Hiking" interest but user did not have uploaded "Hiking" photo and (2) uploading "Hiking" photo turns out to be successful to the cohort where the user belong.

4. User behaviour
  - swipe history
  - smartphoto score (Tinder model output)

5. Technical flags
  - is smartphoto feature enabled
  - is selfie verified

User behaviour features are used to infer causal relationship between "Dating Concepts" and "Dating Outcomes".

### Management
Source data are manageed by MatchGroup DLP (Data Lifecycle Policy), and we uses only available data from them and remove after research or demo in 3 months.
We are planning to use user data of English speaking geos like AUS, NZL, CAN, and US excluding EU/ EEA/ Switzerland/ US-Illinois.
We did March demo and preliminary researches with New York City data.

## 2. LLM with prompts
Those data are passed to LLM APIs in three different scenarios.
1. Research to figure out what common concepts (e.g., curly hair) exist and how much they contribute to dating outcomes (e.g., rSRR) -> the cumulative result of the research is called "Knowledge Base".
2. Demo to provide photo review (e.g., add photos with sunglasses cause you mentioned it as your interest) to specific user. -> the result of the demo is called "Review", "Coaching", or "Advice".
3. Evaluation on input prompt and output review to check if the pair matches our criteria by LLM-as-a-judge.

We use multiple LLM API vendors while research, demo, and evalutaion including
- claude
- chatgpt
- gemini

### Research and training
While research, we have leveraged AI agents to produce statistical report based on human hypothesis. The agent understands user's request and run SQL to Tinder databricks to confirm the given hypothesis; e.g., What makes them Kings/Queens?, Score difference between bathroom tile background vs cafe background, How fatal are sunglasses, masks, smoking, or cropped ex-partner photos?, Backfire effect when an Introvert user posts party photos.
The agent consists of tens of predefined prompts from how to access Tinder data infrastructure to how to write up reports. They are just playbooks how to do research in general with some terminology explained. See: `https://github.com/matchgroup-ai/aura-agent/tree/main/.claude/skills`
Currently, the user (engineer) is responsible for safe usage of LLMs when giving "User Prompt" to the agent. The prompt must restrict search space of the agent not to extract social attribute or report sensitive data to our "Knowledge Base".
Same data is used to train custom model (1) to identify domain concepts from the profile elements of the given user and (2) to calculate causal effect of adjusting profile elements to find out best advice. The list of domain concepts is confirmed and fixed by human after identification not to include concept with legal concern.

### Demo and evaluation
On demo, we will use custom model to figure out what domain concepts are found and where to coach for given user and LLM API to generate the coach in natural language. The user only can access to generated coach. See: `2026-04-08-aura-demo-prompt.md`

On evaluation, we will use LLM-as-a-judge to evaluate the coach result based on Eval criteria. Specific wording for each criteria would be updated daily from below baseline.
- Accuracy: Does the insight correctly identify the most important gap in the profile?
- Framing: Is the title correctly framed according to the content strategy?
- Visual proof: Is the insight supported by curent state of the profile photo?
- Title/Benefit Cohesion: Does the insight and the stated benefit logically connect and remain appropriate given the available profile information?
- Accuracy: Is the recommended action accurate given the available profile information? (e.g., System is not interring, only using explicitly available information on the profile)
- Appropriateness: Is the recommended action appropriate? (i.e. Although user mentioned weed in their bio, it does not ask the user to upload weed smoking photo)
- Ease: Is the recommended action easy and realistic for the user to execute?
- Title/Action Cohesion:Does the action logically follow from the identified insight?
- Diversity score: Is the Low/Medium/High diversity classification correct based on the photo set?
