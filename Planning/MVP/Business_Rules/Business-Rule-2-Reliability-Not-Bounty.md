# Business Rule #2 — Reliability, Not Bounty

**Product:** WHO
**Status:** Draft v1
**Applies to:** No-show accountability, deposits, and the reward system

---

## Executive Summary

WHO needs a way to discourage flaking on matched activities without recreating the incentive problems that come from a naive "deposit gets forfeited to whoever shows up" model. That original design gave one stranger a financial reason to want another to fail — bad for trust, bad optics, and it pressures people into showing up to potentially unsafe situations just to avoid losing money to someone else.

This rule replaces that with a system built on **reputation and shared reward, not punitive payout to an individual.** No user ever profits from another user's no-show. No-show consequences are about accountability (reliability standing) and restitution to the community (a shared reward pool), never a bounty paid to a specific person — and critically, the platform itself is never positioned to profit from flaking either, since that just moves the same bad incentive up one level.

---

## How It Works

### 1. No-show fees never go to another user
If a user no-shows a confirmed activity without cancelling in time, any fee charged does not go to the other attendee(s). This eliminates the "I benefit when you fail" dynamic entirely — no user has a financial reason to want anyone else's date to fall through.

### 2. How a no-show is determined (show-up verification)
Every consequence in this rule — fees, reliability impact, reward eligibility — depends on the app knowing, reliably, whether someone actually showed up. Verification is based on **mutual in-app check-in within a geofence**, not hardware (NFC tags, printed QR codes). A static code can be photographed and reused by someone who was never there; two independent people checking in from their own devices, near each other, in a tight time window, is much harder to fake without active collusion between the missing parties. No physical infrastructure is required, so it works at any venue from day one.

**Two-stage arrival window.** For an activity scheduled at time T:
- **Stage 1 — first arrival:** anyone in the matched group has a fixed window from T (e.g. 30 min) to be the first to check in. If nobody checks in before this expires, the activity never started, and it's logged as a failed activity rather than pinned on a specific individual (unless only one side had people queued to attend).
- **Stage 2 — the rest of the group:** the clock restarts from the moment of that first check-in. The remaining party/parties have a window from there to check in — roughly 15 minutes for a pairwise (1-on-1) match, roughly 30 minutes for a group, since larger groups plausibly trickle in over a longer span.

**Anyone who doesn't check in within their applicable window is an individual no-show** — this is what actually triggers the fee and reliability consequences described elsewhere in this rule.

**Running-late extensions.** A blanket "keep waiting" option would let someone stall indefinitely at the other party's expense, so extensions are capped and asymmetric depending on who's already arrived:

- **Case A — nobody has checked in yet (both/all late):** symmetric situation, so either side can propose an extension. Because nobody present is being kept waiting, the ceiling can be longer — up to ~60 minutes for a flexible, non-pre-booked activity. Pre-booked activities (a class, a venue reservation with a fixed start) should have a shorter ceiling, since the activity itself has external constraints that don't bend. *(Exact ceiling for pre-booked activities is an open item — see below.)*
- **Case B — someone has already checked in and is waiting on the rest:** asymmetric, since the present party is the one bearing the cost of waiting, so they control it. The late party can send a "running late" request; the present party accepts or denies.
  - **Denied** → final for that window. No second request. Original deadline stands.
  - **Accepted** → the window extends by the requested amount, and a **second** request becomes possible — but only after a cooldown equal to half the just-granted extension has passed (e.g. a 15-minute extension means the late party can't ask again until ~7.5 minutes into it). This allows a genuine "I'm still not quite there" follow-up without enabling back-to-back spam requests.
  - After the second request is resolved (accepted or denied) either way, no further requests — that window is final.

**Group quorum.** For groups (3+), individual check-in status and the group's overall success are tracked separately:
- Every person who checks in gets full reliability credit, regardless of whether the group as a whole reaches quorum. Someone who showed up should never be penalized for other people's no-shows.
- The activity counts as having occurred once the number of checked-in attendees meets the activity's required minimum group size.
- If the checked-in count stays below the minimum once all windows expire, the activity failed to reach quorum. This is logged the same way as Business Rule #1's "didn't reach minimum group size" reason — it's a live, same-day version of that same failure mode, and useful for the same upstream tuning purpose.

**Optional layer:** once quorum is reached (pairwise: both parties; group: minimum met), an in-app "we met" photo prompt can fire as a lightweight, non-mandatory tiebreaker for the rare disputed case — not a requirement for verification to succeed.

### 3. No-show fees don't become platform profit either
Fees collected from no-shows are not treated as company revenue. They flow into the **Community Voucher Pool** (see below), blended with a small slice of normal platform transaction revenue — deliberately never 100% flake-funded, so the pool's growth isn't visibly tied to other users failing to show up. This keeps the company's own incentives aligned with completed, successful activities (its actual revenue model), not with flaking.

### 4. Cancellation window
Cancelling 24+ hours before a matched activity results in a full refund and no reliability impact. This distinguishes honest, early cancellation from a no-show, and avoids pressuring users into attending activities they're uncomfortable with just to protect a deposit.

### 5. Three separate reliability signals
Each signal has exactly one job. They are not interchangeable and don't feed into each other's outcomes.

| Signal | Timeframe | Purpose |
|---|---|---|
| **Weekly activity record** | Logged every week | Raw data — did the user show up or not, per activity. Feeds the two scores below. |
| **Monthly reliability score** | Resets to zero each month | Determines entries into the Community Voucher Pool for the following month. A fresh start each month means a bad month doesn't lock someone out of future rewards. |
| **Lifetime reliability score** | Never resets, slow-moving | Determines deposit tier (new/unproven users pay a refundable deposit; users with strong long-term standing eventually need none) and, at the extreme, throttling or removal for chronic no-shows. Never affects voucher eligibility. |

This split means a user can be deposit-free based on long-term standing while still having a quiet current month with few or no voucher entries — these are answering two different questions ("do we structurally trust you" vs. "were you reliable this cycle") and are allowed to disagree.

### 6. Community Voucher Pool
- **Funding:** no-show fees blended with a small slice of platform transaction revenue.
- **Entry:** earned only through a user's own positive behavior that month (completing activities, maintaining a reliability streak) — never as a result of someone else's no-show.
- **Distribution:** a hybrid model — a guaranteed entry for the month's top-N most reliable users, with remaining pool capacity distributed via weighted lottery among all qualifying users. This rewards consistency predictably while keeping broad participation and lottery-driven engagement for everyone else.
- **Payout form:** vouchers (discounted or free upcoming activities, partner venue perks) rather than cash — keeps the mechanic product-native and avoids the money-transmitter complexity of a cash payout.
- **Display:** shown to users as their own earned entries and an abstract progress-toward-next-drop indicator, not as a running dollar total sourced from no-show fees. The reveal is tied to the existing weekly ritual (announced alongside the Wednesday match reveal) to reinforce the same habitual check-in rather than compete with it.

### 7. Personal Voucher Builder (opt-in)
A separate, individual mechanic: users can opt into paying a slightly elevated platform fee on their activities, with the difference accruing into a personal, redeemable voucher balance. This is a pure loyalty/store-credit mechanic — it involves no shared pool, no other users, and no incentive complications, since it's a user building credit from their own spending.

### 8. Skin-in-the-Game Tier (kept separate from reliability)
Opting into the elevated fee (Personal Voucher Builder) reflects that a user has chosen to put more financial stake into their own participation — but this is never allowed to influence the reliability score itself. Reliability must stay a pure behavioral signal (did you show up, consistently, over time), since it's the basis for deposit tiering and trust decisions elsewhere in the system. Letting payment buy a better reliability reading would mean trust could be purchased rather than earned, undermining the entire point of Business Rule #2.

Instead, opting into the elevated fee grants a separate, clearly distinct status — a "Committed" or "Invested" tier/badge, shown apart from reliability so users and the matching system both understand it reflects stake, not track record. This tier may function as a **minor tiebreaker** in matching (e.g., between two otherwise-equivalent candidates for a slot), but it can never outrank or substitute for a genuinely strong reliability history. Financial commitment can nudge; it cannot buy its way past someone's actual behavior.

---

## Open Items (not yet resolved)
- Actual deposit amounts and the lifetime-score thresholds that move a user between deposit tiers.
- The elevated fee amount for the Personal Voucher Builder, and whether that balance expires or caps.
- Community Voucher Pool: exact blend ratio between no-show fees and platform revenue, the threshold that triggers a voucher drop, and the entry-weighting formula (flat per-activity vs. streak-multiplied).
- Size of the guaranteed-floor (top-N) tier relative to overall pool size — intended to be tuned with real usage data rather than set now.
- Exact mechanics of the Skin-in-the-Game tiebreaker — how "otherwise-equivalent" is defined, and how much weight it carries relative to other soft-ranking signals (e.g. group-size closeness from Business Rule #1).
- Exact extension ceiling for Case A (both/all late) on pre-booked activities — the flexible-activity ceiling (~60 min) is set, but the shorter pre-booked number isn't yet.
