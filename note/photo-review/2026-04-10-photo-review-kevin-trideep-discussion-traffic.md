# Kevin's report
This assumes we would trigger a model run on a daily basis, either on a schedule or upon the first app open on a new day, not a new run on each app open / no duplicates on the same day for one user.
Take a look and let me know what you think. Based on this forecast we'll have one very high day of demand at 100% of DAU (unless we ramp) and then take about 3 weeks to reach a steady state of 7%.

There are two milestones where we need to forecast the QPS for the Photo Review:
1. The experiment
2. The rollout

We can use the same method for both, scaling down the global population to the experiment population as a % of MAU for those geos.

The total QPS will be:
1. New logins (100% on day 1, and fewer new logins each day, converging to the new regs and reacts)
2. Users who edit their Profile daily is a constant.

The analysis below shows (based on a sample of 1000 users) that ~4% is the expected rate an active user makes any of the followings change to their profile:
1. Add or Delete a Photo
2. Change Bio
3. Change Prompt
4. Add or delete Interest
5. Add or delete Descriptor

There will be some overlap in the % of new logins and % of profile updates, so this will be a partial overestimate.

Profile Changes that Trigger a Refresh
1. Add Photo
2. Delete Photo
3. Change Bio
4. Chane Prompt
5. Add Interest / Descriptor

# Discussion
Kevin) I took a stab at the estimate for QPS. I modeled it here as a % of daily active users, which we could use for our test regions or global rollout (~18M = 100%).

Trideep) I believe this would much higher upper threshold. This assumes, the users have changed their photos every single day.
The cost would be roughly: 18 million × $0.01 = $180,000/day

Kevin) actually no - it only triggers an execution the first time someone logs in during the window (starting at 100% of DAU when no one has received the score) and quickly ramping down. If you log in again with no change to your profile you don’t get rescored. There is a 4% profile update rate daily we add to this, meaning the stable state would be 7%.
So 180k * .07 = 12.6k would be the global stable state.
There is some overlap as well (new login with profile change) that would make this an overestimate

Trideep) I see. So basically 180k first day post that 12.6k per day ?

Kevin) If you look at the mode you can see the ramp schedule yeah - and this is for global
We would also probably exclude some users who didn’t meet the minimum profile criteria or already had good profiles
