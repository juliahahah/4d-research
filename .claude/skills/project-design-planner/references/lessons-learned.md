# Lessons learned

A running log of pitfalls hit while using this skill — so the same mistake, deferred requirement, or missed safety check doesn't recur on the next project. Entries accumulate across projects; this file is generic to the skill, not project-specific (project-specific facts still belong in `.claude-design/knowledge.md` / `style.md`).

## When to read this

- **Phase 1 (clarify).** Before finalizing clarification questions, skim entries whose category matches the current request (e.g. anything CI/automation-shaped). A past lesson here often turns into a question or a pinned-down parameter now, instead of a mid-implementation surprise later.
- **Phase 3 (self-review).** When reviewing `design.md` like a PR reviewer, check whether any entry's "how to avoid next time" applies to this plan and hasn't been addressed.

## When to write here

After implementation or review turns up something `design.md` didn't anticipate — a deferred requirement that cost a review round, a bug that slipped past self-review, a safety gap — add an entry **before** moving on. Keep it short: what happened, why, what concrete step would have caught it. Most recent first.

---

## Entry template

```
### <short title> (<project or context>, <date>)
**What happened:** ...
**Root cause:** ...
**How to avoid next time:** ... (ideally: which Phase/step this should feed into)
```

---

### Recurring cleanup automation: long tail was deferred requirements, not bugs

**What happened:** Building a recurring cleanup automation took many more review rounds than expected. In hindsight, ~80% of the iteration was deferred requirements + review rounds + safety re-validation — not defects. Only one change was a real bug (a `set -e`/`pipefail` abort inside a piped command substitution), and self-review caught it before merge.

**Root cause:** Phase 1 didn't pin down all the tunable parameters for the automation up front, so schedule, thresholds, scope, and notification format each surfaced one at a time during later rounds.

**How to avoid next time:** For any recurring job / automation / CI change, settle these in Phase 1 in one pass: schedule/frequency; the threshold(s) and what they measure; scope (env/region/namespace); the notification channel *and* its content+format (plain text vs. rich card — the biggest source of churn); and the safety posture (dry-run vs. direct action, exemptions). *(This is already codified as a standing rule in Phase 1 of SKILL.md — kept here as the concrete incident that motivated it.)*
