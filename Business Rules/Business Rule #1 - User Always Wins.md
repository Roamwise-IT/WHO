# Business Rule #1 — User Always Wins

**Product:** WHO
**Status:** Draft v1
**Applies to:** Activity recommendation, pruning, and matching flow

---

## Executive Summary

Once a user has done the work of narrowing their weekly recommendation pile down to a final ranked shortlist, WHO guarantees they walk away matched into **something on that list** — never nothing, and never silently substituted without explanation.

The user is never told "you'll get your #1 choice." They're told the app will try, and if it can't, they'll always land on one of the other options they *already said they wanted*, with a clear reason why the higher choice fell through. The failure mode this rule eliminates is a user investing real effort (browsing, comparing, ranking) and ending the week with nothing to show for it. That outcome is never acceptable under this rule — every completed prune-and-rank flow ends in a real, scheduled activity.

This is the trust foundation the rest of the matching system is built on: users prune honestly and rank sincerely because they know the process pays off.

---

## How It Works

### 1. Inputs to this stage
By the time this rule applies, a user has already:
- Received their weekly recommendation pile (activities matched to their profile — intensity, talk level, duration, group size range, romantic↔platonic scale)
- Pruned the pile down via head-to-head comparisons to their final shortlist (2–3 activities)
- **Ranked** that shortlist in order of preference

### 2. Matching runs once, on the full batch, on a fixed weekly timeline
Matching does not run per-user or per-activity as people finish pruning. It runs once, after the prune window closes for the full cohort, on a fixed weekly cycle. This is required for the guarantee to be honest — the app can't promote someone to their #2 choice unless it knows, at that same moment, who else's overlapping choices could still fill that slot.

The weekly cycle is fixed and identical for every user:

| Day | Event |
|---|---|
| **Sunday** | Weekly recommendation pile released to all users |
| **Sunday – Tuesday night (midnight)** | Prune-and-rank window is open |
| **Tuesday, midnight** | Selections lock — nothing can be changed after this point |
| **Wednesday night** | Matching runs on the full batch; confirmed activity matches are released to users |
| **Thursday – Sunday** | Matched activities take place |
| **Rest of week** | No new selections can be made until the next Sunday release |

This makes WHO a weekly ritual rather than an anytime-browsing app — a deliberate point of differentiation from swipe-based dating apps, where activity is constant and low-commitment. Users have one deadline to make a real decision, one reveal moment to look forward to, and a fixed window in which their matched activities happen. It also guarantees the matching engine always has the full cohort's rankings before it runs, which is what makes the Business Rule #1 guarantee possible in the first place.

**Open flag:** the Tuesday-midnight cutoff needs a timezone anchor — either a single platform-wide cutoff time or each user's local midnight — to be decided.

### 3. Matching attempts top-down through each user's ranking
For each user, the system attempts to place them into their #1 ranked activity first, checking for overlap with others whose pruned lists include the same activity, subject to:
- The activity's minimum group size (floor of 2, higher for some activity types)
- The user's stated group-size ceiling
- The user's ideal group size and how much importance they placed on hitting it (see group size mechanic, below)

If placement succeeds, done. If not, the system logs **why** (see reason library below) and attempts placement into the user's #2 choice, then #3.

### 4. Reason transparency
Every downgrade comes with a stated reason, drawn from a fixed set rather than freeform text — this keeps messaging honest, consistent, and easy to audit:
- Didn't reach minimum group size
- Group size preference couldn't be matched closely enough
- Time slot conflict
- Venue/activity capacity reached

Users are never simply moved to a different activity without being told which of these applied.

### 5. The guarantee
By design, a user who completes the prune-and-rank flow will be placed into **one of their final 2–3 choices**. WHO does not promise the #1 choice — it promises an honest attempt at #1, transparent reasons if that fails, and a guaranteed landing on the list the user themselves built.

### 6. Group size interaction
Each user's group-size preference has four parts: a fixed floor of 2, a stated ceiling, an ideal number, and an importance weight on hitting that ideal. Importance is treated as:
- **Above a threshold:** a hard filter — the system won't place the user in a group outside their ideal ± tolerance, even if it's within their stated ceiling.
- **Below the threshold:** a soft ranking signal — among otherwise-valid groupings, the system prefers the one closest to the user's ideal.

### 7. Data value
Every failed placement and its reason is logged. This becomes the primary signal for tuning the recommendation engine — e.g., if "didn't reach minimum group size" is the dominant failure reason, that indicates cohorts are too small or the pile-generation step is too scattered, and is a direct input to fixing upstream matching, not just this stage.

---

## Resolved: Cohort Cadence

The entire user base runs on one universal fixed weekly cycle (see timeline in section 2) — there is no rolling or per-cohort prune window. A user joining mid-week simply waits for the next Sunday release. This was chosen deliberately to reinforce WHO as a weekly ritual and to keep the Business Rule #1 guarantee possible (matching requires the full cohort's rankings at once).

**Remaining open flag:** timezone handling for the Tuesday-midnight cutoff (see section 2).
