# Business Rule #3 — Intent, Not Identity

**Product:** WHO
**Status:** Draft v1
**Applies to:** Overlap-matching — the platonic↔romantic scale and gender preference

---

## Executive Summary

Once users overlap on a pruned activity (per Business Rule #1) and clear the group-size mechanic, WHO still needs to decide whether the *specific people* who overlap actually belong in the same group. Two signals govern this: the **platonic↔romantic scale** (what kind of energy someone wants from the people they end up with) and **gender preference** (who they want in the room, regardless of why).

These are two genuinely different questions, and this rule exists to keep them from getting tangled. A user's romantic/platonic setting should never silently gate or override their gender preference — a fully platonic user is just as entitled to a same-gender-only activity ("girls' night," "guys' trip") as a romantically-open user is entitled to filter by who they're interested in dating. **Intent is about what kind of connection someone's open to. Identity preference is about who they want there. Neither one explains or excuses the other.**

---

## How It Works

### 1. Platonic↔romantic scale: single point, automatic tolerance
Each user sets a single point on a platonic↔romantic slider — no manually configured range. The app automatically builds a tolerance band around that point to determine who they're compatible with, so the UX stays a simple slider while the matcher still has room to work with.

The band is **not uniform across the scale**:
- **Narrower near the platonic extreme** — someone who sets themselves fully platonic likely means it literally ("just friends," not "mostly friends"), so their effective matching range should stay tight.
- **Wider toward the middle** — sitting in the middle of the scale is itself a signal of openness, so the band can reasonably stretch further there.

Exact band widths and the curve between them are a tuning question (see Open Items), but the *shape* — tight at the platonic extreme, wider in the middle — is decided now since it affects the data model.

### 2. Pairwise matching
For a 1-on-1 match, two users are compatible on this axis if their tolerance bands overlap. This is a straightforward binary check once band widths are set.

### 3. Group matching: cohesion, not internal pairing
For group activities (3+), WHO does not attempt to pair off individuals within the group — that's a different, more invasive feature than activity-based matching is meant to provide. Instead, this functions as a **cohesion filter**: every member's tolerance band needs to overlap within an acceptable spread of the group's overall range. A user whose point sits well outside where the rest of the group clusters shouldn't be placed there, even if every other criterion (activity, group size, timing) matched. The exact spread threshold for "acceptable" is a tuning question (see Open Items).

### 4. Gender preference: independent, always-on
Gender preference is **not gated by, or derived from, the platonic↔romantic scale**. It's a standalone, categorical filter that sits alongside other hard preferences (budget, group size, etc.) — same tier, same treatment, applied at the overlap-match step regardless of where a user sits on the romantic/platonic axis.

This matters because "platonic" does not mean "indifferent to who's in the room." A fully platonic user may still want a same-gender-only activity for reasons that have nothing to do with romance (a girls' night, a guys' trip). The matcher doesn't need to know *why* someone set a gender preference — it's simply honored, the same way any other stated preference is.

### 5. Scope for MVP: categorical filter, not composition control
At this stage, gender preference is a simple filter (e.g., "match me with women," "match me with any gender") — not a weighted group-composition target (e.g., "at least half the group should be women"). Composition-style controls are a meaningfully harder matching problem — they turn gender from a per-person filter into an optimization constraint across the whole group — and are deliberately deferred rather than built into MVP (see Open Items).

---

## Open Items (not yet resolved)
- Exact tolerance-band widths and the curve between the platonic extreme and the midpoint (narrower-to-wider is decided; the actual numbers are not).
- Exact group-cohesion spread threshold — how far a group's members' bands can diverge before someone is excluded from that group.
- Weighted/composition-based group gender controls (e.g., "at least half women") — deferred past MVP; revisit once there's real group composition data to justify the added matching complexity.
