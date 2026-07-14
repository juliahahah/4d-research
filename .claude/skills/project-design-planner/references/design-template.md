# design.md template

The review-ready implementation plan. This is what the user reads before deciding to implement. It must be concrete and faithful to the real code — including before/after diffs.

Use this structure:

```markdown
# Design: <short title of the change>

_Date: <date>. Based on knowledge.md and style.md in this folder._

## Requirement summary
The request, restated precisely. Include clarifications received from the user and the **concrete, measurable goal** (what must be true when done, and how it's verified).

## Assumptions
Every default chosen instead of asking the user. If a reader disagrees with one, they know exactly what to correct.

## Approach
The intended solution and *why it fits this project*. Reference concrete existing patterns/modules from knowledge.md and conventions from style.md. If alternatives were considered, note the tradeoff and why this one won.

## Files to change
| File | Action | Reason |
|------|--------|--------|
| path/to/file | new / modify / delete | one line |

## Detailed changes (before/after)
For each modified file, show a faithful diff of the relevant section:

### path/to/file.ext
```diff
- existing line as it really is in the file
+ proposed replacement
```
For NEW files, show the full proposed content in a fenced block.
Diffs must match the actual code read in Phase 2 and conform to style.md. Keep them **minimal** — only the lines that must change; no drive-by refactors or reformatting. Do not fabricate line content — if unsure, re-read the file first.

## Commit plan
How to split the work into commits, matching the commit convention from `style.md` (message format and granularity). Default is Conventional Commits (see `conventional-commits.md`) unless the history shows otherwise. Example:
- `feat(scope): <description>` — files/scope
- `fix(scope): <description>` — files/scope

## Risks & open questions
- What could break, edge cases, migration/backfill concerns
- Anything the user should weigh in on before implementation

## Testing plan
How the change should be verified (which tests to add/update, how to run them), consistent with the project's test setup.

## Out of scope
What this plan deliberately does not address.
```

Keep the diffs the star of the document — they're the reason the user can trust the plan without reading the whole codebase themselves.
