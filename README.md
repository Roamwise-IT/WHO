# WHO

**Status:** Planning — docs only, no tech stack chosen yet.

## What is WHO?

WHO is an activity-based blind dating app. Instead of swiping on profiles, users set preferences — activity intensity, group size, talkativeness, duration, budget, and a platonic↔romantic scale — and get matched with others who want the same kind of activity. The activity is the unit of matching, not the person.

Each week, users receive a recommendation pile, prune it down via head-to-head comparisons, and rank their final shortlist. Matching runs once per week on the full cohort, and every user who completes the process is guaranteed to land on one of their ranked choices — never nothing, and never silently substituted without explanation (see [Business Rule #1](<Planning/MVP/Business_Rules/Business-Rule-1-User-Always-Wins.md>)).

The weekly cadence — recommendations released Sunday, prune/rank window closes Tuesday midnight, confirmed matches revealed Wednesday night, activities happen Thursday–Sunday — is a deliberate ritual, differentiating WHO from always-on swipe apps like Tinder, Bumble, and Hinge.

No-show accountability is built on reputation and shared reward rather than a payout to another user or the platform itself — see [Business Rule #2](<Planning/MVP/Business_Rules/Business-Rule-2-Reliability-Not-Bounty.md>) for the full reliability/deposit/voucher-pool design.

## Repo Structure

```
WHO/
├── README.md                  ← this file
├── Planning/
│   └── MVP/
│       ├── Business_Rules/
│       │   ├── Business-Rule-1-User-Always-Wins.md
│       │   ├── Business-Rule-2-Reliability-Not-Bounty.md
│       │   ├── Business-Rule-3-Intent-Not-Identity.md
│       │   └── Business-Rule-4-Conduct-Not-Reliability.md
│       └── WHO-System-Architecture.md
└── (code lives here once a stack is chosen)
```

- **`Planning/<version>/`** — one folder per product version (`MVP`, `V1`, `V2`, ...). As the product evolves, each version's folder holds the *full active set* of business rules at that point — rules carry forward into later versions rather than being duplicated only when changed. A rule that's superseded gets revised in place (with a note or version bump), not silently overwritten.
- **`Planning/<version>/Business_Rules/`** — one markdown file per rule. Filenames use a shell/URL-safe pattern — `Business-Rule-N-Short-Title.md` (hyphens, no spaces or `#`) — since spaces and `#` cause quoting headaches in the terminal and break unencoded markdown/URL links. The number is chronological (when the rule was written), not a priority ranking. The human-readable `Business Rule #N — Title` form is used inside each doc's own heading and in the table below, not in the filename.
- Each rule doc follows the same shape: an Executive Summary, a numbered "How It Works" breakdown, and an "Open Items" section for anything intentionally left undecided (numeric thresholds, tuning values, etc.) rather than guessed at.
- **`Planning/<version>/WHO-System-Architecture.md`** — a single living document, sibling to `Business_Rules/`, that translates the business rules into system-level design (services, boundaries, event flow). It grows one "Part" at a time as each business rule is drafted — see below.
- Code, when it starts, will live at the top level of `WHO/` alongside `Planning/`, not inside it — `Planning/` stays reserved for design/spec docs across the life of the project.

## Business Rules So Far

| # | Title | File | Covers |
|---|---|---|---|
| 1 | User Always Wins | `Business-Rule-1-User-Always-Wins.md` | Guarantee that a completed prune-and-rank flow always ends in a real matched activity; the weekly release/prune/match timeline |
| 2 | Reliability, Not Bounty | `Business-Rule-2-Reliability-Not-Bounty.md` | No-show accountability without punitive payouts to other users or the platform; show-up verification; deposit tiers; the Community Voucher Pool and Personal Voucher Builder |
| 3 | Intent, Not Identity | `Business-Rule-3-Intent-Not-Identity.md` | The platonic↔romantic scale and gender preference at the overlap-match step, and why one never gates the other |
| 4 | Conduct, Not Reliability | `Business-Rule-4-Conduct-Not-Reliability.md` | Blocking (personal, silent) vs. reporting (community signal, logged); the trust & safety record kept separate from the reliability score; two-track moderation escalation |

## System Architecture

[`WHO-System-Architecture.md`](<Planning/MVP/WHO-System-Architecture.md>) is the system-design counterpart to the Business Rules above — where a Business Rule defines *what* the product guarantees and *why*, the Architecture doc defines *how that gets built*: service boundaries, event flow between services, and the interfaces each part is and isn't allowed to have with the others.

It's a **living document**, growing one Part at a time as each Business Rule is drafted, rather than being written once at the end. As of now it covers four parts, one per business rule so far:

- **Part 1 — Scheduling & Matching Engine** (from Rule #1): the weekly state machine and the batch-run matching guarantee.
- **Part 2 — Reliability & Voucher Service** (from Rule #2): no-show handling, the three independent reliability signals, and the Community Voucher Pool — exposing only a single "Committed" tiebreaker flag back to matching.
- **Part 3 — Overlap-Compatibility Filter** (from Rule #3): the platonic↔romantic tolerance band and gender preference, kept as two independent filter passes inside the matching pipeline.
- **Part 4 — Trust & Safety** (from Rule #4): blocking and reporting, structurally separate from reliability, exposing only a binary paused/not-paused flag back to matching.

The document also carries a **Cross-Cutting Design Principle** — a discipline that runs through all four parts and every business rule they're derived from: distinct signals must be evaluated on their own terms, and never collapsed into, gated by, or allowed to substitute for one another. Any future Part added to the document gets checked against this principle before being merged in.

**This document isn't final at four parts.** It tracks the Business Rules 1:1, so it stays open-ended until the business rules themselves are considered complete — activity sourcing is the next confirmed rule (see Open Items below), with payments/take-rate, non-no-show cancellations, and an identity/trust baseline flagged as likely follow-ons. Each new rule gets its own Part here, following the same derivation pattern.

## Commit Convention

This repo uses [Conventional Commits](https://www.conventionalcommits.org/): `type(scope): description`

**Types in use so far:**
- `docs` — anything documentation/spec related (which, at this stage, is everything)
- `chore` — repo structure, tooling, non-content changes
- `feat` / `fix` / `refactor` — reserved for once code exists

**Scope guidance:**
- Adding a brand-new rule doc → scope the whole area: `docs(business-rules): add Business Rule #3 - <Title>`
- Editing an existing rule (new section, wording fix, resolving an open item) → scope to that specific rule so its history is easy to isolate later: `docs(business-rule-2): add skin-in-the-game tier section`
- Adding or updating a Part in the System Architecture doc → scope to `architecture`: `docs(architecture): add Part 4 - Trust & Safety`
- Structural changes to the Planning folder itself (e.g. branching a new version) → `chore(planning): add V1 business rules folder`
- Use kebab-case for scopes (`business-rules`, not `business_rules`).

**Examples:**
```
docs(business-rules): add Business Rule #2 - Reliability, Not Bounty
docs(business-rule-2): add skin-in-the-game tier section
docs(business-rule-2): resolve deposit tier thresholds
docs(architecture): add WHO System Architecture, Parts 1-4
chore(planning): restructure business rules by version
```

## Open / Not Yet Decided

These are project-wide open questions, separate from the open items listed inside individual rule docs:

- Tech stack — nothing chosen yet, repo is docs-only at this stage.
- Activity sourcing — who supplies/creates activities in the weekly pile (curated, partner venues, user-submitted, or a mix) — not yet defined; next business rule to be drafted.
- Payments and take-rate, non-no-show cancellations/disruptions (venue closures, weather), and an identity/trust baseline at signup — flagged as likely future business rules, not yet started.
- Trusted-contact sharing and in-activity safety check-ins — deferred to V1/V2, not MVP (see Business Rule #4).

This README will evolve alongside the project — update it when the repo structure changes, a stack is chosen, or a new top-level concept (beyond what's in the Business Rules) gets introduced.
