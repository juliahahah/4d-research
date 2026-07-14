---
name: project-design-planner
description: Turn a feature request or change request into a concrete, review-ready design plan for an existing codebase. Use this skill whenever the user describes something they want built, changed, fixed, or added to a project and expects a plan BEFORE code is written — phrases like "I want to add X", "help me implement Y", "how should I change Z", "plan out this feature", or handing over a requirement/ticket. The skill clarifies vague requirements by asking the user, then actually reads the project (never guessing) to produce knowledge.md (a factual survey of the project), style.md (the project's coding conventions), and design.md (how to implement the request, including concrete before/after code diffs and which files change). This skill ONLY produces planning documents — it never edits the project's code. Use it even when the user doesn't say the word "design" but clearly wants an implementation plan grounded in their real codebase.
---

# Project Design Planner

Turn a requirement into a review-ready implementation plan grounded in the **actual** codebase — not guesses. The user reviews the plan (`design.md`) and only then decides to implement.

## Core principles

1. **Read, never guess.** Every claim in the output must trace to something actually observed in the code, config, or docs. If something can't be verified by reading, say so explicitly rather than inventing it.
2. **This skill only plans. It never edits project code.** The sole deliverables are three Markdown documents. Implementation happens later, after the user approves.
3. **Clarify before diving in — but don't over-ask.** Only ask the user when not asking would risk building the wrong thing or when there are multiple reasonable interpretations. Otherwise pick a sensible default and record it as an assumption in `design.md`. A **concrete, measurable goal** and the user's **commit convention** must be established before producing `design.md` (see Phase 1).
4. **Minimal diff.** The plan changes only what the requirement needs. No opportunistic refactors, renames, reformatting, or unrelated cleanup. Prefer the smallest change that satisfies the goal and fits existing patterns.
5. **Review your own plan like a PR reviewer.** Before handing off, re-read `design.md` critically as if reviewing someone else's pull request — check correctness against the real code, minimality of the diff, adherence to `style.md`, edge cases, and whether the commit breakdown matches the user's convention. Fix what a good reviewer would flag.
6. **Reuse prior surveys.** `knowledge.md` and `style.md` are expensive to produce. Build them once, then reuse them; only refresh when the project has meaningfully changed.

## Output location

Write all three files to `.claude-design/` in the project root:
- `.claude-design/knowledge.md`
- `.claude-design/style.md`
- `.claude-design/design.md`

Create the folder if it doesn't exist. If the user specifies a different location, honor that.

**These are local planning artifacts — they must never be committed or pushed.** Ensure `.claude-design/` is git-ignored: if the project has a `.gitignore` and it doesn't already list `.claude-design/`, append it (add a line `.claude-design/`). If there's no `.gitignore`, create one containing that line. Do this as part of Phase 2. This is the one file outside `.claude-design/` this skill may touch — and only to add the ignore entry, nothing else.

---

## Workflow

Follow these phases in order. Don't skip the clarification phase, and don't skip reading the project.

### Phase 1 — Clarify the requirement

Read the user's request carefully. Extract what's already clear from the conversation first (requirements, referenced files, constraints, corrections). Then decide whether you need to ask.

**Always establish these two before writing `design.md` (ask if not already clear):**
- **A concrete, measurable goal.** Not a vague description — what specifically should be true when this is done, and how it's verified. If the user gave only a fuzzy ask, pin this down before proceeding.
- **The commit convention.** How commits are normally made (message format, commit granularity, whether one PR = one commit or several). **Observe this from the real git history first** (`git log`, see Phase 2) rather than asking — consistent with "read, never guess." **Default to Conventional Commits** (`references/conventional-commits.md`) when the history is empty, inconsistent, or doesn't clearly follow another convention; if the history clearly follows a different style, match that instead and record it in `style.md`. Only ask the user if you genuinely can't infer a clear convention. This shapes how `design.md` breaks the work into commits.

**For a recurring job / automation / CI change, pin the tunable parameters up front — in one pass.** Each parameter left open tends to surface mid-implementation and cost another code → review → validation round. Settle at least: schedule / frequency; the threshold(s) and what they measure; scope (env / region / namespace); the **notification channel *and* its content + format** (plain text vs. rich card — easy to under-specify and the biggest source of churn); and the safety posture (dry-run vs. direct action, exemptions).

**Check `references/lessons-learned.md` for past pitfalls relevant to this request** (e.g. anything CI/automation-shaped) before finalizing clarification questions — a past lesson often turns into a question or a pinned-down parameter now, instead of a mid-implementation surprise later.

**Ask the user when:**
- The concrete goal or commit convention above is still unclear.
- The request has multiple reasonable interpretations that would lead to materially different designs.
- Key acceptance criteria are missing (what does "done" look like?).
- Scope is ambiguous (one screen vs. whole flow; one endpoint vs. a subsystem).
- There's an implied constraint you can't verify (target users, performance, backward compatibility, deadline-driven tradeoffs).

**Don't ask when:**
- The answer is already in the conversation or trivially inferable from the codebase.
- A reasonable default exists — use it and note it under "Assumptions" in `design.md`.

Keep questions few and high-leverage. Prefer offering concrete options over open-ended questions. Confirm your understanding in one or two sentences before moving on.

### Phase 2 — Survey the project (produce or reuse knowledge.md and style.md)

First ensure `.claude-design/` is git-ignored (see Output location). Then check whether `.claude-design/knowledge.md` and `.claude-design/style.md` already exist.

- **If they exist:** Read them. Do a quick freshness check (see `references/freshness-check.md`). If the project hasn't meaningfully changed, reuse them as-is. If it has, update only the affected sections and note what changed.
- **If they don't exist:** Produce them now using the survey method below.

**Survey method — structure-first, then targeted depth (never guess):**

1. **Map the structure.** List the directory tree (ignore `node_modules`, `.git`, `dist`, `build`, `venv`, etc.). Read the top-level signal files: `README`, `package.json` / `pyproject.toml` / `requirements.txt` / `go.mod` / `Cargo.toml` / `pom.xml`, entry points, and config files.
2. **Identify the stack, architecture, and common commands** from what you read — languages, frameworks, build tooling, test setup, how the app is layered, and the actual build/run/test/lint commands (from README, package.json scripts, Makefile, or CI config — not guessed).
3. **Deep-read only what's relevant to the request.** Follow imports and references from the entry points into the modules the requirement touches. Read those files fully. Don't attempt to read every file in a large project — that wastes context and isn't needed. **For large codebases, dispatch parallel sub-agents** to explore different modules/areas concurrently, then consolidate their findings — rather than reading everything serially in the main context.
4. **Read the commit history.** Run `git log` (e.g. the last 30–50 commits, `git log --oneline -50` plus a few full messages) to infer the project's real commit convention: message format (Conventional Commits vs. free-form), typical granularity, and whether commits are squashed. Record this in `style.md`. If there's no history or it's inconsistent, note that and ask the user in Phase 1.
5. **Record uncertainty honestly.** If part of the project is relevant but you didn't read it, say so in `knowledge.md` rather than guessing at its behavior.

See `references/knowledge-template.md` and `references/style-template.md` for the exact structure of each document. `style.md` must prioritize existing linter/formatter config (eslint, prettier, ruff, black, gofmt, .editorconfig, etc.) as ground truth, and only infer conventions from code where no config dictates them.

### Phase 3 — Produce design.md

With the requirement clarified and the project understood, write `design.md`. This is the document the user reviews. It must follow the template in `references/design-template.md` and include:

- **Requirement summary** — the request restated precisely, plus any clarifications received, including the concrete measurable goal.
- **Assumptions** — every default you chose in place of asking.
- **Approach** — how you'll implement it, and *why* this approach fits the project's existing patterns (reference specifics from `knowledge.md` / `style.md`). Favor the **minimal diff**: no refactors or cleanup beyond what the goal requires.
- **Files to change** — a table of each file, whether it's new/modified/deleted, and a one-line reason.
- **Concrete before/after diffs** — for each modified file, show the relevant existing code and the proposed replacement as a unified diff or clear before/after blocks. New files show their full proposed content. These must be consistent with the real code you read and with the project's style. Keep them minimal — only lines that must change.
- **Commit plan** — how to split the work into commits, matching the commit convention from Phase 1 (message format and granularity). When that convention is Conventional Commits, follow `references/conventional-commits.md` (type/scope/description, `!` or `BREAKING CHANGE:` footer for breaks, lowercase type, one logical change per commit). Do not append tool-generated trailers (e.g. `Co-Authored-By:`) unless the user asks for them.
- **Risks & open questions** — anything that could break, edge cases, things worth the user's attention before implementing.
- **Testing plan** — how to verify, consistent with the project's test setup.
- **Out of scope** — what this plan deliberately does not do.

Keep diffs faithful to the actual code. If you didn't read a file closely enough to write an accurate diff, read it now — do not fabricate line content.

**Then self-review like a PR reviewer.** Before handing off, re-read the finished `design.md` critically as if it were someone else's pull request: is every diff correct against the real code? Is the diff truly minimal, or did unrelated changes sneak in? Does it follow `style.md`? Are edge cases and error paths handled? Does the commit plan match the user's convention? Also check `references/lessons-learned.md` for any entry whose "how to avoid next time" applies here and hasn't been addressed. Revise anything a careful reviewer would flag, and note any residual concerns under Risks.

### Phase 4 — Hand off

Present the three files to the user (use a file-presentation tool if available). Briefly summarize the approach and flag the top risk or open question. Do **not** start editing project code — that's a separate step the user initiates after reviewing.

**When the user later initiates implementation:** you may edit code and commit on a feature branch, but **never `git push` or open a PR without the user's explicit confirmation for that specific push.** Always present the committed change and ask before pushing, every time — approval to implement or to commit is *not* approval to push. The user reviews locally first and decides when it goes up.

**If implementation or review turns up something `design.md` didn't anticipate** (a deferred requirement that cost a review round, a bug that slipped past self-review, a safety gap), add an entry to `references/lessons-learned.md` before moving on — see that file's template. This is what keeps the same pitfall from recurring on the next project.

**Opening a PR for the RD team:**
- Title must indicate the target branch, e.g. `[master] Chore/riskai image 1.26.2`.

**CI / automation changes** — apply whenever the change is a workflow, scheduled job, or
anything that runs unattended, especially if it mutates or deletes resources. Reading code
can't prove the real auth/cluster/API path, and an unattended job's only witness is its output —
so:

1. **Prove it for real, once, before it can auto-run.** A workflow gated on `schedule` /
   `workflow_dispatch` won't fire from a feature branch — temporarily add a push trigger to test it:
   ```yaml
   on:
     push:
       branches: ["**", "!master"]   # every branch except master
   ```
   Run it once, confirm the job goes green **and** any outward effect happened (e.g. the
   notification was sent), **screenshot both into the PR**, then delete the temporary trigger
   before merge — master must never carry an unvalidated, self-triggering (especially
   destructive) automation.
2. **Design for observability.** Emit a human-readable summary of what it did and why, to both
   the run log and the notification; capture command errors into that report (not a bare
   "failed"); name states unambiguously so a 2 a.m. failure is diagnosable from output alone.
3. **Attach evidence, not just the diff** — reviewers usually can't run your branch. Green run +
   summary, delivered notification, resulting state, each with a one-line "this shows X".
4. **Reply to every review comment, including bots (e.g. Copilot)** — fix it or say why not; an
   unresolved thread blocks merge. Remind the user to paste PR review comments back to you (you
   can't read PR threads yourself) before addressing them.

**Operating GitHub Actions:** Actions tab → pick workflow → open the run (match by branch +
trigger) → job → step logs; the **Summary** panel shows `$GITHUB_STEP_SUMMARY` output. Manual
trigger: "Run workflow" → pick branch → Run. **Never click "Cancel workflow"** — can leave
resources half-provisioned. On a failed run, read the log first; escalate to the workflow owner
per team SOP if you can't resolve it.

---

## Reference files

- `references/knowledge-template.md` — structure for `knowledge.md`
- `references/style-template.md` — structure for `style.md`
- `references/design-template.md` — structure for `design.md`
- `references/freshness-check.md` — how to decide whether to reuse or refresh a prior survey
- `references/conventional-commits.md` — Conventional Commits 1.0.0, the default commit convention
- `references/lessons-learned.md` — running log of past pitfalls; check in Phase 1/3, append in Phase 4
