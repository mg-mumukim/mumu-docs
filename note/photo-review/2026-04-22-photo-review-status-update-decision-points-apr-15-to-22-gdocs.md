| Key           | Value                                                                                          |
| ------------- | ---------------------------------------------------------------------------------------------- |
| Source        | https://docs.google.com/document/d/1VPSzigsFo83AvBCvKO911Efwz11IrVDyrHtbPnZn1Eg              |
| Downloaded at | 2026-04-22 14:36:24                                                                              |

---

Documenting key updates and decisions (made and upcoming) from the past week to get Jaini up to speed and keep MG AI & Tinder aligned

## **Evaluation Iteration 3 results (on repetitiveness & diversity)**

### What changed:

* **Diversity category update:** merged Interests & Lifestyle into one, and kept Activities separate to reduce ambiguity


| Category | Interests & Lifestyle | Activities |
| :---- | :---- | :---- |
| **Definition** | Photos that give visual cues about tastes, interests, preferences, or personal identity / Passive or low-action moments that still reveal something meaningful about who they are or what they like | Photos that capture the person actively engaged in a clear, specific behavior or event. Observable, intentional actions (ideally time-bound) Physical or skill-based engagement Moments that demonstrate doing rather than being |
| **Examples** | Sitting in a cafe with a book (from your list) Travel photo with cultural or scenic context (from your list) At a coffee shop patio with a pet Having a picnic at the park At a farmer’s market holding produce or walking through stalls Walking a dog in a neighborhood or park | Running a marathon Mid cooking Playing guitar on stage Rock climbing, surfing, dancing |

* **Repetitiveness definition**: Refined to better capture framing repetition in close-up selfies (as opposed to flagging almost literally identical photos)  
  ![image1](images/279d16692f2e.png)

## **Next evaluation plans**

| Eval Round | Date | Evaluation Area |
| :---- | :---- | :---- |
| 4 | Week of Apr 22 | Gemini 3.1 Flash Lite vs Pro comparison (to validate whether Lite can reach comparable quality- we need this for cost optimization) |
| 5 | Week of Apr 28 | Coaching evaluation based on the updated AURA Knowledge Base |
| 6 | Week of May 5 | Coaching voice & tone (copy & framing) |

## **Other related timelines**

* **Week of Eval 4**: MG AI to complete estimation/ MVP engineering doc to share async with SOUP engineering  
* **Week of Eval 5**: (Juniper visiting Palo Alto 4/27\~4/29) sync with SOUP engineering to review implementation feasibility

## **Key MG AI ML updates and their implications**

1. **Custom ML model descoped from MVP**  
* **Decision**: Custom ML-model based approach will not be included in MVP Photo Review.  
* **Reason**: Current model output is not stable enough to meet quality bar within timeline, showing no significant benefit when compared with Gemini-pro model based version that the team has been evaluating on the web demo. Waiting for further development would block downstream decisions such as serving implementation.  
* In short, we’re no longer gating product/ eng decisions on ML readiness.

2. **Leveraging ML research findings to update the Knowledge Base**  
* However, some ML research findings can still be utilized to build better Knowledge Base (currently used to find the best possible action to recommend user in Photo Review)  
* This means MVP will still be powered by Tinder data-driven analysis.

3. **Deciding between two different serving strategies**  
* **Document WIP:** [Photo Review MVP Engineering Docs](https://docs.google.com/document/d/1Q4x0DwGJuDGjTaqFAtlbHFH-KD1cWZosPI-zdhFTQ9U/edit?tab=t.zbd36zecf8rg)  
* The team has to essentially choose between:  
  * Option 1: Full batch (precomputed coaching)  
  * Option 2: Async on trigger (currently takes 30 to 60 seconds to generate coaching on trigger)  
* Since the tradeoff is between UX immediacy and infra simplicity/cost, product opinion is required here.

4. **Decision timeline**  
* ML research finding Knowledge Base integration to be completed by **4/24,** to be ready for Evaluation iteration 5\.  
* Final decision on serving approach to happen also by **4/24**, so that it could be shared with SOUP backend (eng) ahead ot the time to get their opinion


## **Other sync & decisions pending**

* Need conformation with SOUP eng if this timeline still somewhat holds true or we should expect any major changes [(DRAFT) SOUP Q2 2026 Engineering Capacity](https://docs.google.com/spreadsheets/d/1VuchpwGRfXE3hKMotGcMf4is75tvAsLu-S6ZeZnLoUQ/edit?usp=sharing)

## **Recap on remaining to-do**

- [ ] [Juniper Han](mailto:juniper.han@match.com) to share the updated web demo for Eval iteration 4 and new guide for the team (latest by 4/23)  
- [ ] MG AI team to complete and share the [Photo Review MVP Engineering Docs](https://docs.google.com/document/d/1Q4x0DwGJuDGjTaqFAtlbHFH-KD1cWZosPI-zdhFTQ9U/edit?tab=t.zbd36zecf8rg) so that all relevant engineers and stakeholders are informed about the serving options and its implications  
- [ ] MG AI team to update Knowledge Base by end of 4/24 to be ready for Eval iteration 5  
- [ ] [Juniper Han](mailto:juniper.han@match.com) to sync with Jaini on the new repetitiveness/ diversity definition and update the PRD  
- [ ] [Juniper Han](mailto:juniper.han@match.com)to walk [Jaini Shah](mailto:jaini.shah@gotinder.com)through the serving options (if needed) and answer any questions  
- [ ] [Jaini Shah](mailto:jaini.shah@gotinder.com)to update Juniper & team on any design updates if any  
- [ ] [Jaini Shah](mailto:jaini.shah@gotinder.com)to update Juniper & team on SOUP timeline update if any  
- [ ] Evaluators to do the round 4 evaluations to see where Gemini flash model may be underperforming when compared with Gemini pro model
