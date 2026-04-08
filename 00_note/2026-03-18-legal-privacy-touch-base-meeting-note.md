# AURA x SOUP team stream
Goal: MG AI team walks away from this meeting knowing exactly what to share with Kelsey by when

Sharing product context (to cover any gaps still existing after legal sync with SOUP team)
E.g., List of data being used and potential risk assessment in the AURA system

Three specific components to gather preliminary legal feedback on data handling:
Component,Function,Privacy Context
AURA Demo,Profile-based Coaching,Currently on hold to align with the latest Tinder Photo Review UI/UX.
AURA Research Agent,Knowledge Base Generation,Accesses high volumes of user profile data to build the "brain" of the agent. Shared infrastructure with the Demo.
ML Training & Serving,Tinder Photo Review,Uses the same raw data as the Research Agent but involves distinct processing pipelines and storage locations.

Given the context above, let’s define exact documents/ information Kelsey needs and set “send by” dates for MG AI to ensure this doesn’t bottleneck Photo Review launch
-> ASAP because don't want to break things at too late time (agree and understood)

One small additional question: what is the right "cadence" or “size/ nature of the project” for us to fill the Privacy by Design form?
-> Legal requirement: any time where collecting or new data is required/ using existing data for new purposes
-> We can determine things like (v2 shares same document with v1 or separated docs needed ..;.) together
-> Yes to Kelsey’s point it’s very dependent on the business use case and if it’s materially different from what we’ve done in previous experiments.

# Action items
- List of data that is being used in the Knowledge Base + what kind of insights we are looking for from the Knowledge Base + if the data is being moved outside of the KB, where it goes
  - Fill out Privacy by Design for KB and Research Agent (One to cover both) + include the list of experiments already planned out in relevance to this stream (+ AURA demo for it)
  - Fill out Privacy by Design for Photo Review (ML Server)
- Double check if LLMs does not extract biometric data from user - identifying duplicated photo is okay but you cannot do this by recognizing "person"
