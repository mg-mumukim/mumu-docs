2026-04-13 | MGAI x Tinder ML AURA Sync
Attendees: Juniper Han Trideep Rath Ben Shin (shurain) Sungjoo Ha Jaini Shah Mumu Kim Claude Yoon Brandon Choi

Agenda
Mumu Kim Tinder Privacy by Design (PBD) for AURA demo on Profile Coaching/ Waiting for legal meeting (tomorrow)
Mumu Kim Profile Feedback - MGAI x Tinder/ Cost estimation
If you have any questions on numbers, please leave comments on the cost estimation document!
Single-number cost summary per (region x serving option)
designed to provide a quick answer to 'so how much does it cost roughly?' rather than detailing ALL available options and parameters
organized assumptions and decisions made for this topic
Scope
Option
LLM Cost / Month
Plan
GPU Cost / Month
Total
Test regions (USA+CAN+AUS)
Batch
$33K
OD
$50
$33K
Test regions  (USA+CAN+AUS)
Real-time
$66K
SP
$4K
$70K
Global
Batch
$163K
OD
$247
$163K
Global
Real-time
$325K
SP
$4K
$329K

Conclusion
Only English region + Batch inference ($33K/mo) fits within $50K budget for now
GPU node cost is negligible in batch mode + on-demand because Qwen3-VL-8B is small enough to run batch jobs in few hours if no real-time requirements are given
Mumu) 1-2hrs for batch jobs in English-using regions // 5hrs globally -> because we don't have to run GPU nodes for 24 hours using on-demand was cheaper.
Ben) nit: I think it's okay to apply the same discount for all ec2 as we have a bulk deal - you can ask our MLSEs
Trideep) [A] 79 profiles/sec seems too high throughput based on previous experience = need re-estimation; [B] we need to confirm upside of adding Qwen to our pipeline; [C] how much tokens will be saved after adding Qwen to our pipeline
Mumu) IMO we can benchmark (not ball park estimation) and evaluate (by comparing pure Gemini v.s. causality-aware Qwen + Gemini) them  after implement this to AURA demo
Discussion
mumu) Global traffic-based cost estimation may be premature; beyond-English deployment is our long-term goal, but the timing and values are unclear, so sizing MVP (and/or right next milestone) costs against 100% global traffic seems too conservative
MVP scope is English regions (US+CAN+AUS+NZL); AFAIK already agreed, considering traffic, language coverage, and legal constraints
Cost cuts would hurt the experiment; at MVP stage, meeting a budget cutoff likely means degrading LLM quality, which undermines the whole point of validating whether the feature is worth building
Juniper) This is good guess for now
Brandon) I'm not sure; need time
Details
https://ai-demo.dev.matchgroupcentral.net/aurademo/demo/cost-dashboard is for interactive version of cost estimation to compensate the document, but the evidence and logic used for the estimate are diverged (dashboard is outdated); Mumu Kim will update it
Brandon Choi Serving options depending on user scenario
Maybe a mix of option 2 and 1 would be the best?


Option 1
Option 2
Option 3
Approach
Gemini Batch API (Precompute)
Gemini API (Near-real-time precompute) - using 1m model
True Realtime (<5s)
User scenario
1. User opens app + is eligible for batch
2. Server queues user for batch job
3. No result when visiting Profile Home (post Day 0)
4. Result generated later
5. User returns via next session or push/nudge
6. Sees “AI review ready”
1. User triggers event (app open, photo update, etc.)
2. Server starts to  precompute immediately
3. Result may not be ready on immediate visit
4. Ready within seconds to minutes
5. User re-engages via push/nudge if latency is longer than 5 seconds
6. Views result on return
Dependent on model development

1.  User enters Profile Home
2. Loading state shown if user enters before coaching is ready
3. Result generated within 2–5 seconds
4. Feedback shown in same session
User Experience
Fully async experience

Results may already be available, but are not guaranteed (depends on prior batch processing)

Feels clean if timing works, but can feel disconnected
Faster async than batch

Results are typically not available immediately, but are expected shortly after (in-progress computation)

Still not instant; may require revisit to Profile Home
Most intuitive and seamless

Immediate feedback at point of intent

Best UX if performant


Best Case
User returns next day

Result is ready

Seamless “ready for your review” moment
User navigates to Profile Home after short delay

Result is ready or almost ready
Feels timely and relevant
Result loads in ~2–3 seconds

Minimal perceived waiting time

High product quality experience


Worst Case
User never returns to the app
Result goes unseen

Photos change → result becomes stale
User checks Profile Home  immediately → no result

Requires re-entry, which may feel like a broken / interrupted flow
If loading unfortunately takes >5 seconds

Loading feels long/blocking

Worse experience than async if slow

What is the missing information we need to make the call here?
Engineering ) We need to understand batch options -> 24 hours + how true is this
Brandon) Don’t we already know this though?
Trideep) first question is how much does Gemini really take (in terms of hours- we’re saying it’s somewhere between 0 and 24, but can we have a more realistic/ narrower estimate?) Also, I read somewhere they can take 200k/hr when we batch, let’s make sure this is true. Let’s talk with the Gemini team. I’m saying this because with NanoBanana we had a problem with satisfying 1 QPS at the moment. They haven’t figured out scaling from their side.
Brandon) Okay, makes sense. Do you know if that’s a problem with Gemini flash?
Trideep) I hope not, but let’s make sure
Trideep) For option 3, have we thought about using a faster/ lighter model like Gemma?
Brandon) I think we should try Gemini flash light for option 3 because it has the fastest inference time (gemini 2.5 flash light)
Trideep) So there’s no difference between number of calls between Option 2 and 3
Trideep) One thing I observed with Nanobanan is that removing thinking didn’t change anything (lol)
Product) Are we okay with this from a product standpoint?

