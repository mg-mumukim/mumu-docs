# #ml-photo-review Serving Options Thread (2026-04-16 01:30 KST)

**Source:** Slack #ml-photo-review thread  
**URL:** https://matchgroup.enterprise.slack.com/archives/C0AMBBQGU6R/p1776270600309899  
**Thread timestamp:** 2026-04-15 16:30 UTC / 2026-04-16 01:30 KST  
**Participants:** Mumu Kim, Trideep Rath (cc Brandon Choi, Juniper Han, Ben Shin, Biggie Choi)  
**Note:** Full thread captured via Glean on 2026-04-16. Total 8 messages.

---

## Mumu Kim (16:30 UTC)

Hi Trideep.Rath Ben.Shin cc Brandon.Choi Juniper.Han and Biggie.Choi

Following up from our sync on 4/13, we identified the three engineering uncertainties that need to be resolved before we can confidently compare the serving options. Brandon.Choi is leading the effort to benchmark and write down reports to the document. Engineering details might be changed while investigation or benchmark.

**Question:** Once we have confidence on these three items, is there anything else on the engineering side that would hinder the Option 1/2/3 decision?

Our downstream work (e.g., backend contract refinement, dummy server testing, task planning, ...) depends on this decision. If our direction is different from what you expect, or if Trideep.Rath or Ben.Shin need additional inputs, we'd like to know now so we can prepare it instead of them.

1. Gemini Batch API behavior
2. Qwen3-VL L40S performance
3. Cost reduction from Qwen3-VL + Gemini pipeline

Referenced document: https://docs.google.com/document/d/1Q4x0DwGJuDGjTaqFAtlbHFH-KD1cWZosPI-zdhFTQ9U/edit?tab=t.zbd36zecf8rg#heading=h.6femou6ts1ys

---

## Mumu Kim (16:31 UTC)

ps. I can't join today's meeting because I'm out of office in the morning (KST). Brandon may cover technical qnas.

---

## Trideep Rath (22:37 UTC) — re: processing lag estimate

Our current estimate is 1-6 hours for English regions, based on a prior 10K-request backfill (Joel.Ham's chat tag extraction pipeline work for Recs pillar). Good to know. Thanks.

---

## Trideep Rath (22:45 UTC) — re: Gemini Batch / lag

Is product okay with lag? Looks like there is a 24-48 hour lag. I thought hourly batch would solve this problem, please correct me if I am wrong. I also had a word with Jaini.Shah and it seems it is fine.

---

## Trideep Rath (22:45 UTC) — re: batch serving architecture

Yes this should work. Mostly likely should be possible but needs backend confirmation, but need to understand how this would work for both backfilling and steady state. I guess steady state is for each hourly window, we take in all the users who have add/updated their photo and create a batch.

---

## Trideep Rath (22:45 UTC) — re: ML/backend ownership

Also let's also discuss the exact system for this with the backend and what components should the ML team own and what should be owned by Backend. What are our options.

---

## Trideep Rath (22:50 UTC) — re: cache cost reduction (item 3)

Does it really provide the cash benefit? The portion of system prompt is just 3% of the entire context window because most tokens are used to represent each user's profile and thinking process, so I think cache cost reduction is negligible. Not just the cache aspect but the overall aspect. If we call at profile-home, we are only calling for users who visit the profile home, that's the cheapest. But this is only valid if the feedback only exists at profile home and not when we want to have notifications or bottom sheet. For batch, we are basically processing for all users who might not even open the profile home or not even see the feedback. Let's try to understand the cost tradeoffs.

---

## Trideep Rath (22:56 UTC) — re: Qwen3-VL L40S performance (item 2)

On this, this only is valid if we are able to say that having this intermediate step substantially improves the output quality. Let's first prove the value of it before adding this as a step. If the team strongly feels this is the right direction, we should design the system in such a way that the current overall system we are discussing is able to fit in when the tech is valuable.

---

## Brandon Choi (23:00 UTC)

Juniper.Han Mumu.Kim I'm not invited to the meeting

---

## Mumu Kim (2026-04-16 08:02 UTC)

Trideep.Rath Thank you for detailed and insightful feedbacks! I also expect you and Brandon had talked with those topics in the meeting.

Brandon.Choi I expect refined problems and requirements has to be documented here right after preparation of Eval 4. Expected outcome is ... refined version of this slack message is written in google docs. Do I aligned with you?
