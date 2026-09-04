# Business Rule #4 — Conduct, Not Reliability

**Product:** WHO
**Status:** Draft v1
**Applies to:** Blocking, reporting, and moderation escalation

---

## Executive Summary

WHO needs two different safety mechanisms that must not be confused with each other: a way for an individual to unilaterally remove someone from their own experience (**blocking**), and a way for the community to flag genuine conduct concerns to WHO itself (**reporting**). These answer fundamentally different questions and carry different weight — blocking is a private, no-consequence personal boundary; reporting is a public-facing safety signal that lives on a person's record and can trigger real moderation action.

This rule also establishes that conduct concerns are tracked entirely separately from the reliability score defined in Business Rule #2. Reliability answers "did this person show up, consistently." Trust & safety answers a completely different question: "has this person's conduct raised concern." Merging them would dilute genuine safety signals into noise, or wrongly imply that punctuality says anything about safety (or vice versa).

---

## How It Works

### 1. Blocking — personal, unilateral, silent
- Any user can block any other user at any time, for any reason or none.
- Blocking removes the blocked user from the **blocker's own future matching pool only**. It has no effect on anyone else's matching, no effect on the blocked user's reliability or trust & safety record, and the blocked user is never notified.
- If a block happens after the two are already matched for the current week, the blocker is also pulled from that pending match — treated the same as an honest early cancellation under Business Rule #2, with no reliability penalty.
- No reason is required, and nothing beyond the block itself is logged.

### 2. Reporting — a community signal, not a personal action
Reporting is different in kind from blocking: it isn't just removing someone from your own pool, it's flagging a concern to WHO about the person's conduct, and it goes on their record.

- Reports are filed against a defined set of categories rather than freeform text alone — e.g., harassment, felt unsafe / safety threat, inappropriate behavior, fake profile — consistent with the fixed reason libraries already used in Business Rule #1 (downgrade reasons) and Business Rule #2 (quorum-failure reasons), for consistency and auditability.
- Every report is logged to the reported user's trust & safety record, regardless of category.

### 3. Trust & safety record — a fourth signal, structurally separate from reliability
- This is **not** the reliability score. It is its own signal, tracking conduct concerns rather than show-up behavior.
- It is invisible to other users — never shown as a rating, never something to "match against." It exists purely to inform moderation decisions.
- A user's reliability score and trust & safety record are allowed to tell completely different stories about the same person, and that's by design — a perfectly punctual user could still raise safety concerns; a chronically late user is very likely not a safety risk. Conflating the two would misrepresent both.

### 4. Escalation: two tracks based on severity
- **Standard categories** (minor conduct issues, disputes, etc.): a single report enters a human review queue without any automatic action against the reported user. Multiple reports, from multiple distinct reporters, escalate to an automatic pause on matching, pending review.
- **Severe categories** (explicit safety threats, violence, harassment involving threats): a single report is enough to trigger an immediate automatic pause on matching, pending the same human review. This asymmetry is deliberate — briefly pausing someone unfairly during review is a far smaller cost than leaving a genuine threat active in the matching pool.
- In both tracks, the pause is provisional, not a final judgment. Human review determines the actual outcome — reinstatement, warning, extended suspension, or permanent removal.

### 5. Deferred to V1/V2 (not MVP)
Flagged here so they aren't lost, not because they're unimportant:
- **Trusted-contact sharing** — sending your matched activity's time, place, and who you're matched with to someone outside the app.
- **In-activity safety check-in** — a lightweight in-app prompt during or after the activity to confirm everything is okay.

---

## Open Items (not yet resolved)
- Exact multi-report threshold that triggers auto-pause for standard categories (how many reports, over what timeframe, from how many distinct reporters).
- The full, finalized list of severe vs. standard report categories — the split in principle is decided, the complete category list is not.
- What reinstatement actually requires after a provisional pause is lifted (e.g., does a formal warning get issued, does anything reset).
- Appeals process for a user who believes a report against them was made in bad faith.
