# Conventional Commits 1.0.0 (慣例式提交)

The default commit convention this skill assumes when planning a `Commit plan`, unless the
project's real `git log` clearly follows a different one (see "Precedence" below).

Source: https://www.conventionalcommits.org/zh-hant/v1.0.0/ — licensed CC BY 3.0.

## Message structure

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

- **type** — REQUIRED. A noun such as `feat`, `fix`, followed by an OPTIONAL scope and a
  REQUIRED `: ` (colon + space) before the description.
- **scope** — OPTIONAL. A noun in parentheses describing the section of the codebase, e.g.
  `fix(parser):`.
- **description** — REQUIRED. Short summary of the change, right after the `: `.
- **body** — OPTIONAL. Free-form, may span multiple paragraphs. MUST start one blank line after
  the description.
- **footer(s)** — OPTIONAL. Start one blank line after the body. Each footer is a token, then
  `: ` or ` #`, then a value (git-trailer style). Footer tokens MUST use `-` instead of spaces
  (e.g. `Reviewed-by`, `Refs`), with `BREAKING CHANGE` as the only exception.

## Types

- `feat:` — adds a feature → maps to SemVer **MINOR**.
- `fix:` — fixes a bug → maps to SemVer **PATCH**.
- Others allowed (from `@commitlint/config-conventional`, Angular-based): `build:`, `chore:`,
  `ci:`, `docs:`, `style:`, `refactor:`, `perf:`, `test:`. These have no implicit SemVer effect
  unless they carry a breaking change.
- Extra types are permitted; they don't bind SemVer.

## Breaking changes → SemVer MAJOR

A breaking change is signalled either way (a commit of *any* type can be breaking):

- **`!` before the colon**: `feat!:` or `feat(api)!: ...`. If `!` is used, the
  `BREAKING CHANGE:` footer MAY be omitted and the description should describe the break.
- **Footer paragraph**: `BREAKING CHANGE: <description>` — token MUST stay uppercase,
  followed by `: ` and a description. `BREAKING-CHANGE` is treated as identical.

## Examples

```
docs: correct spelling of CHANGELOG
feat(lang): add polish language
feat!: send an email to the customer when a product is shipped
```
```
fix: prevent racing of requests

Introduce a request id and a reference to latest request. Dismiss
incoming responses other than from latest request.

Reviewed-by: Z
Refs: #123
```
```
feat: allow provided config object to extend other configs

BREAKING CHANGE: `extends` key in config file is now used for extending other config files
```
```
revert: let us never again speak of the noodle incident

Refs: 676104e, a215868
```

## Key rules (MUST)

- Every commit MUST be prefixed with a type; `feat`/`fix` MUST be used for their meaning above.
- The `: ` after type/scope is REQUIRED; the description MUST follow it immediately.
- Body MUST begin one blank line after the description; footers one blank line after the body.
- Units are case-insensitive **except** `BREAKING CHANGE`, which MUST be uppercase.
- Type casing (upper/lower) is free but SHOULD be consistent — this skill uses **lowercase**.

## Precedence for this skill

1. If the project's `git log` already follows a clear, consistent convention, match **that** —
   "read, never guess" (Core principle 1). Record it in `style.md`.
2. If history is empty, sparse, or inconsistent, default to Conventional Commits above and note
   the assumption in `design.md`.
3. When splitting work, prefer one logical change per commit (FAQ: split commits that satisfy
   more than one type). Keep commit messages clean — do not append tool-generated trailers
   (e.g. `Co-Authored-By:`) unless the user asks for them.
