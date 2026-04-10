# Document draft
## TL;DR
[!NOTE] Give answer to readers who want to know exact numbers of LLM cost based on traffic assumption.

## About this document
This document outlines the cost estimation for each execution scenario. Those scenarios are to support decision-making for backend-ML system design, test planning, and infrastructure settings.

## LLM cost per request
We use numbers for `gemini/gemini-3.1-flash-lite-preview` for our further estimation. It costs roughly $8.6K per million requests, without batch API discount.
In practice, the length of prompts, the number of photos, the size of photos, and the number of output tokens might vary depending on the model, user distribution, and our iterations but we assume that the change in cost will not be significant.

See [Gemini 3.1 Flash-Lite: Built for intelligence at scale](https://blog.google/innovation-and-ai/models-and-research/gemini-models/gemini-3-1-flash-lite/) for model catalogue.

See [AURA Cost Dashboard](https://ai-demo.dev.matchgroupcentral.net/aurademo/demo/cost-dashboard) for a detailed breakdown of parameters affecting LLM cost per generation.

## Traffic quantities
This section describes how we estimated the number of LLM calls in first stage and steady stage.

[!NOTE] Refine this table.
```csv
Country,DAU
Global,18M
USA,2.8M
CAN,450K
AUS,400K
```

[!NOTE] Please refer other documents and fill this (Traffic quantities) section clearly. (R1) Use Kevin's number if Ben's number conflicts with Kevin's. (R2) Explain step-by-step logic and rational of the calculation which should be understood well by high school students. (R3) Use DAU number from above table.

## Serving scenario
This section describes multiple scenario candidates that are the main exemplar for cost estimation.
See "Serving Optimization" tab for futher engineering details.

[!NOTE] There are two different serving options. (O1) real-time review generation. (O2) batched (24h) review generation and retrieval. Explain each option briefly without introducing too much implementation details. But rational should be rigorous based on previous LLM costs and traffic quantities. Precise numbers and formulations should be represented. Gemini batch API discount is 50%.

## Assumptions
[!NOTE] update this assumption definition and concrete numbers with other documents (discussion, estimation)
### Potentially active user
It is impractical to backfill photo reviews for all users due to the cost and usage limits of 3rd party LLM APIs. I don't know how we can implement this logic, but if we assume we can predict which user will open the app this month with 100% accuracy, then the expected size of the potentially active user base would match the average MAU. Because we need 100% certainty, more "on-the-fly" photo reviews must be computed.

[!NOTE] update this assumption definition and concrete numbers with other documents (discussion, estimation)
### Effective profile change
For example, if the photo review ML server does not rely on the user's name, then a name change should not trigger a profile change event that necessitates a photo review. There are three discussion points: First, should we even care about this topic? Second, are we able to implement this branching logic in estimation phase and production? If we cannot differentiate them in practice or expected cost reduction of differentiation is low, then we can assume all profile change events are effective.

[!NOTE] please add additional assumptions we made if exists.
