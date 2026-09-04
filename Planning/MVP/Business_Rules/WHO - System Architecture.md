# WHO — System Architecture

**Product:** WHO
**Status:** Draft v1 — living document (Parts 1–4 of N)
**Applies to:** Overall system design, service boundaries, and data flow between business rules

---

## Executive Summary

WHO's system architecture is organized around a single weekly cycle, with two core rule sets governing what happens inside it: **Business Rule #1 (User Always Wins)** governs matching, and **Business Rule #2 (Reliability, Not Bounty)** governs accountability after a match is made. They are designed as separate services with a deliberate, narrow interface between them — not because they're unrelated, but because keeping them decoupled is what prevents the incentive problems each rule was written to solve.

The **Matching Engine** runs once per week, in a batch, after every user's prune-and-rank window closes. It guarantees that any user who completes that flow lands on one of their own top choices — never nothing, never a silent substitution. This batch design isn't incidental; it's structurally required, since the engine can only honor someone's #2 choice if it knows, in that same moment, who else's rankings could fill the resulting gap.

The **Reliability Service** governs what happens after a match: no-shows, cancellations, and reward eligibility. Its core architectural constraint is that no user-to-user financial mechanic is allowed to exist — a no-show's fee never routes to the other attendee, and it never routes to platform profit either. It flows into a shared Community Voucher Pool instead, so no party in the system (peer or platform) ever has a financial incentive to want someone else to fail.

These two services touch at exactly one point: a lightweight "Committed" tier signal that the Reliability Service exposes to the Matching Engine as a **minor tiebreaker only** — never a hard filter, and never able to outrank an actual reliability history. This boundary is intentional and load-bearing: it stops financial stake from being able to buy its way past genuine behavioral trust, which is the same failure mode that sank the original (rejected) deposit design.

A third service, the **Overlap-Compatibility Filter**, sits inside the Matching Engine's placement step (between "candidates share an activity and clear group-size" and "placement is confirmed"). It governs whether the specific people who overlap on an activity actually belong in the same group, using two signals that are kept structurally independent: a platonic↔romantic tolerance band and a categorical gender preference. Neither is allowed to gate or explain the other — a fully platonic user is just as entitled to a same-gender-only activity as a romantically-open user is entitled to filter by dating interest. This is another instance of the same architectural discipline running through the whole system: distinct signals are evaluated on their own terms, never collapsed into or overridden by one another.

A fourth service, **Trust & Safety (Blocking, Reporting, and Moderation)**, is the system's fourth independent signal — structurally separate from the Reliability Service (Part 2) even though both concern user behavior. Reliability answers "did this person show up, consistently"; Trust & Safety answers "has this person's conduct raised concern." The two are never merged, never displayed against each other, and never allowed to explain one another — a chronically late user is not treated as a safety risk, and a perfectly punctual user can still raise genuine safety concerns. This service also introduces the system's one genuinely private, unilateral mechanic (blocking, which touches no shared state and generates no record) alongside its one genuinely public, escalating mechanic (reporting, which writes to a permanent record and can trigger automatic matching pauses) — and keeps those two firmly distinct from each other as well.

**Open architectural decisions** — flagged across the four source rules and still needing resolution before implementation — include the timezone anchor for the weekly lock cutoff, deposit amounts and lifetime-score tier thresholds, the exact funding blend ratio for the Voucher Pool, the precise weighting of the Skin-in-the-Game tiebreaker relative to other soft-ranking signals like group-size closeness, the tolerance-band widths and group-cohesion spread threshold that the Overlap-Compatibility Filter depends on, and the multi-report auto-pause threshold and finalized severe-vs-standard category list for Trust & Safety.

---

## Part 1 — Scheduling & Matching Engine
*(derived from Business Rule #1 — User Always Wins)*

### 1.1 Role in the system
This is the system's central clock. It owns the weekly state machine that every other service — including the Reliability Service — synchronizes to, rather than running on its own independent schedule.

### 1.2 Weekly state machine

| Day | State | Emits |
|---|---|---|
| Sunday | Pile released | `pile_released` |
| Sun–Tue midnight | Prune-and-rank window open | `selection_updated` (per user, until lock) |
| Tuesday, midnight | Selections lock | `prune_locked` |
| Wednesday night | Matching runs on full batch | `match_run_started` → `match_confirmed` (per user) / `placement_failed_with_reason` |
| Thu–Sun | Matched activities occur | `activity_occurred` / `activity_no_show` / `activity_cancelled` |
| Rest of week | Locked, no new selections | — |

This event list is the primary interface other services (notably Reliability) subscribe to — they should never need to poll or infer state from timestamps alone.

### 1.3 Matching engine core logic
- Runs once, on the full cohort batch — never per-user or streaming — because promoting a user to their #2 choice requires knowing, at that same moment, whether anyone else's overlapping list could still fill the resulting slot.
- Attempts placement top-down through each user's ranked shortlist (#1 → #2 → #3).
- Placement is constrained by: activity minimum group size, user's stated group-size ceiling, and the user's ideal-size/importance-weight pair.
- Group-size importance is evaluated as a **hard filter** above a defined threshold, and a **soft ranking signal** below it.
- Every failed placement attempt writes a reason code from a fixed library (min group size not reached, group-size preference unmatched, time slot conflict, venue capacity reached) — never freeform text.

### 1.4 Guarantee boundary
The engine guarantees landing on *one of* the user's final 2–3 ranked choices. It does not guarantee the #1 choice, and it never silently substitutes an activity outside that shortlist.

### 1.5 Inputs accepted from other services
- **From Reliability Service:** a single boolean/enum "Committed" tier flag, consumed only as a tiebreaker between otherwise-equivalent candidates for a slot. This is the only inbound dependency the Matching Engine has on Rule #2 — it cannot affect hard filters, minimum group sizes, or the top-down ranking attempt order.

### 1.6 Open architectural flags
- Timezone anchor for the Tuesday-midnight lock: single platform-wide cutoff vs. per-user local midnight — **unresolved**.

---

## Part 2 — Reliability & Voucher Service
*(derived from Business Rule #2 — Reliability, Not Bounty)*

### 2.1 Role in the system
Downstream consumer of the Matching Engine's activity lifecycle events. Owns everything that happens *after* a match is confirmed: attendance outcomes, reliability scoring, deposits, and reward distribution. Deliberately excluded from influencing matching itself, beyond the single tiebreaker signal described in 1.5.

### 2.2 Design constraint driving this service
No mechanic in this service may allow one party — user or platform — to financially benefit from another user's no-show. This was a direct correction to an earlier design (deposit forfeiture paid to the attending user), which created a "root against the other person" dynamic. The architecture must preserve this constraint as new features are added, not just at launch.

### 2.3 Event inputs (subscribed from Matching Engine / activity layer)
- `activity_occurred`
- `activity_no_show`
- `activity_cancelled` (with timestamp, to evaluate against the 24-hour window)

### 2.4 Cancellation handling
Cancellations logged 24+ hours before the activity start: full refund, zero reliability impact. This is evaluated purely off the `activity_cancelled` timestamp relative to scheduled start — no manual review step implied.

### 2.5 Reliability scoring — three independent signals

| Signal | Reset cadence | Feeds | Does not feed |
|---|---|---|---|
| Weekly activity record | Every week | Both scores below | — |
| Monthly reliability score | Resets monthly | Voucher Pool entry eligibility | Deposit tier |
| Lifetime reliability score | Never resets | Deposit tier, throttling/removal | Voucher Pool eligibility |

These are modeled as separate stored values with separate write paths — never derived from one another — since the two downstream consumers (deposit tiering vs. voucher eligibility) are intentionally allowed to disagree.

### 2.6 Community Voucher Pool
- **Funding source:** no-show fees blended with a minority slice of platform transaction revenue — never 100% flake-funded, by design.
- **Entry mechanism:** driven only by the user's own positive monthly behavior; never triggered by another user's failure.
- **Distribution model:** guaranteed entries for a top-N tier of the month's most reliable users, plus weighted lottery entries for the remaining qualifying pool.
- **Payout form:** vouchers/perks, not cash — this also sidesteps money-transmitter licensing exposure that a cash payout path would introduce.
- **Reveal timing:** synced to the existing `match_confirmed` Wednesday event from Part 1, rather than run on its own schedule — reuses the habitual check-in instead of competing with it.

### 2.7 Personal Voucher Builder (opt-in, individual)
A self-contained loyalty mechanic: user opts into an elevated platform fee, the delta accrues to a personal voucher balance. No shared pool, no cross-user effects — isolated from the rest of this service's shared-pool logic.

### 2.8 Skin-in-the-Game tier — the firewall
Opting into 2.7 grants a separate "Committed"/"Invested" status flag — architecturally distinct from the reliability scores in 2.5. This flag is the *only* piece of state this service exposes to the Matching Engine (see 1.5), and it must never be allowed to write into or influence the reliability scores themselves. This is the same anti-corruption boundary described in 2.2, reapplied one layer down: financial stake can nudge a tiebreak, but cannot purchase a better trust reading.

### 2.9 Open architectural flags
- Deposit amounts and the lifetime-score thresholds that move a user between deposit tiers — unresolved.
- Elevated fee amount for the Personal Voucher Builder, and whether that balance expires/caps — unresolved.
- Voucher Pool funding blend ratio, drop-trigger threshold, and entry-weighting formula (flat vs. streak-multiplied) — unresolved.
- Size of the guaranteed-floor (top-N) tier relative to total pool size — intended to be tuned post-launch with real usage data, not fixed now.
- Exact weighting of the Skin-in-the-Game tiebreaker relative to other soft-ranking signals (e.g., group-size closeness from Part 1) — unresolved.

---

## Part 3 — Overlap-Compatibility Filter
*(derived from Business Rule #3 — Intent, Not Identity)*

### 3.1 Role in the system
This filter sits inside the Matching Engine's placement pipeline, one step after Part 1's group-size and overlap checks have already narrowed a slot down to candidates who share a pruned activity and clear the size mechanic. Its job is to decide whether the *specific people* in that overlap actually belong together — it does not select candidates on its own and does not run independently of a placement attempt in progress.

### 3.2 Two independent signals, never merged
- **Platonic↔romantic tolerance band:** derived automatically from a single slider point each user sets — no manually configured range. The band is narrower near the platonic extreme and wider toward the midpoint, since sitting in the middle is itself a signal of openness.
- **Gender preference:** a standalone categorical filter (e.g., "match me with women," "any gender"), applied at the same tier as other hard preferences like budget and group size.

These two signals must be stored, computed, and applied as separate filter passes. Neither is permitted to gate, derive from, or explain the other — the system does not need to know *why* a gender preference is set (romantic interest, a "girls' night," or anything else) in order to honor it.

### 3.3 Pairwise evaluation
For a 1-on-1 candidate pair: compatible on the romantic/platonic axis if their tolerance bands overlap at all. Binary check, evaluated after band widths are configured (see 3.6).

### 3.4 Group evaluation — cohesion, not pairing
For groups (3+), this filter does **not** attempt to pair individuals within the group — that is out of scope for activity-based matching. Instead it runs as a cohesion check: every member's tolerance band must fall within an acceptable spread of the group's overall range. A candidate whose point sits well outside that cluster is excluded from the group even if every other criterion (activity, size, timing) matched.

### 3.5 Gender preference scope for MVP
Implemented as a categorical filter only — not a weighted composition target (e.g., "at least half the group should be women"). Composition-style controls are a materially harder constraint-optimization problem across a whole group rather than a per-candidate filter, and are explicitly out of scope until there's real group-composition usage data to justify the added complexity.

### 3.6 Open architectural flags
- Exact tolerance-band widths and the narrow-to-wide curve between the platonic extreme and the midpoint — shape is decided, numbers are not.
- Exact group-cohesion spread threshold for how far members' bands can diverge before exclusion.
- Weighted/composition-based gender controls — deferred past MVP.

---

## Part 4 — Trust & Safety (Blocking, Reporting, and Moderation)
*(derived from Business Rule #4 — Conduct, Not Reliability)*

### 4.1 Role in the system
This service owns two mechanisms that must be architecturally distinct from each other, and both distinct from the Reliability Service in Part 2: **blocking** (a private, unilateral, no-consequence personal action) and **reporting** (a public-facing safety signal with a permanent record and possible moderation action). It also owns the **trust & safety record** — a fourth signal alongside the weekly/monthly/lifetime reliability signals from Part 2, tracking conduct rather than attendance.

### 4.2 Blocking — no shared state, no record
- Any user can block any other user, for any reason or none, at any time.
- Effect is scoped entirely to the blocker's own future matching pool — the Matching Engine (Part 1) must exclude the blocked user from that one user's candidate set only. No other user's matching, no reliability score, and no trust & safety record are touched.
- If a block occurs after a match is already confirmed for the current week, it triggers the same pull-out handling as an honest early cancellation under Part 2 (Rule #2) — no reliability penalty.
- The blocked user is never notified, and nothing beyond the block relationship itself is stored.

### 4.3 Reporting — a permanent, categorical record
- Reports are filed against a fixed category set (harassment, felt unsafe / safety threat, inappropriate behavior, fake profile, etc.) rather than freeform text — consistent with the fixed reason-code libraries already used in Part 1 (downgrade reasons) and Part 2 (quorum-failure reasons).
- Every report writes to the reported user's trust & safety record, regardless of category — this record is a separate data store from the reliability scores in Part 2.
- The trust & safety record is never surfaced to other users and never used as a matching input — it exists purely to inform moderation, and has no interface into the Matching Engine (unlike the single Committed-tier tiebreaker exposed by Part 2).

### 4.4 Escalation logic — two severity tracks
- **Standard categories:** a single report enters a human review queue with no automatic action. Multiple reports from multiple distinct reporters escalate to an automatic matching pause pending review.
- **Severe categories** (explicit threats, violence, harassment involving threats): a single report is sufficient to trigger an immediate automatic matching pause pending the same human review.
- In both tracks, the pause is provisional — human review determines the actual outcome (reinstatement, warning, extended suspension, permanent removal). The Matching Engine needs only a binary "paused/not paused" flag from this service to act on; it does not need report content or category.

### 4.5 Structural separation from Reliability (Part 2)
Reliability and Trust & Safety are stored, computed, and displayed as fully independent signals. A user's reliability score and trust & safety record are allowed to tell contradictory stories about the same person — a punctual user can still raise safety concerns, and a chronically late user is not treated as a safety risk by virtue of that lateness. No shared computation, no shared display surface, no cross-influence in either direction.

### 4.6 Deferred to V1/V2 (flagged, not built at MVP)
- Trusted-contact sharing (sending matched activity details to someone outside the app).
- In-activity safety check-in prompt.

### 4.7 Open architectural flags
- Exact multi-report threshold for standard-category auto-pause (count, timeframe, distinct-reporter requirement) — unresolved.
- Finalized, complete severe-vs-standard category list — split in principle decided, full list not.
- Reinstatement requirements after a provisional pause is lifted — unresolved.
- Appeals process for reports made in bad faith — unresolved.

---

## Cross-Cutting Design Principle

All four parts of this architecture enforce a shared discipline: **distinct signals must be evaluated on their own terms, never collapsed into, gated by, or allowed to substitute for one another.** Rule #1 protects this at the effort layer (a user's ranking work always pays off, on its own terms). Rule #2 protects it twice — once between users (no-show fees never pay another user) and once between subsystems (financial stake never buys a better reliability reading). Rule #3 protects it at the preference layer (romantic/platonic intent never gates or explains gender preference, and vice versa). Rule #4 protects it at the behavioral layer (attendance reliability and conduct safety are never merged into one score, and a private personal boundary is never conflated with a public safety signal). Any new part added to this document should be checked against this principle — specifically, whether it lets one signal profit from, override, or stand in for another — before being merged in.

---

## Changelog
- **Draft v1:** Part 1 (Scheduling & Matching Engine) and Part 2 (Reliability & Voucher Service) added, derived from Business Rule #1 and Business Rule #2.
- **Draft v1 update:** Part 3 (Overlap-Compatibility Filter) added, derived from Business Rule #3. Executive summary and Cross-Cutting Design Principle updated to reflect all three parts.
- **Draft v1 update:** Part 4 (Trust & Safety) added, derived from Business Rule #4. Executive summary and Cross-Cutting Design Principle updated to reflect all four parts.
