# style.md template

The project's coding conventions. **Config beats inference:** if a linter/formatter config specifies a rule, that config is ground truth — cite it. Only infer conventions from reading code where no config governs them, and mark inferred rules as "(inferred)".

Look for and read, if present: `.eslintrc*`, `.prettierrc*`, `ruff.toml` / `[tool.ruff]` in pyproject, `[tool.black]`, `setup.cfg`, `.editorconfig`, `.flake8`, `tsconfig.json`, `.golangci.yml`, `rustfmt.toml`, `.rubocop.yml`, etc.

Use this structure:

```markdown
# Coding Style

_Derived from config files where available; inferred from code otherwise (inferred items are marked)._

## Tooling / enforced rules
Which linters/formatters are configured and the rules that matter (indentation, quotes, line length, semicolons, import ordering). Cite the config file each rule comes from.

## Naming conventions
- Files / modules
- Functions / methods
- Variables / constants
- Types / classes / interfaces
- Test files
(State whether each is enforced by config or inferred from code.)

## Structure & organization
- How files/modules are organized (feature folders, layers, colocation)
- Where tests live and how they're named
- Import/export patterns

## Idioms & patterns
- Error handling approach
- Async patterns
- State management / dependency patterns
- DRY expectations: what's factored into shared helpers vs. repeated, and where shared code lives
- Comment / docstring conventions

## Commit convention
Inferred from `git log` (message format — Conventional Commits vs. free-form, granularity, squash vs. merge). Cite representative example commit messages. Mark "(inferred)" or note if history was too sparse/inconsistent to be sure. If the history is unclear, the default is **Conventional Commits** (see `conventional-commits.md`) — note that the convention is assumed rather than observed.

## Examples
1-2 short snippets copied from the codebase that exemplify the house style.

## Sources
Config files and representative code files read to produce this document.
```

Prefer concrete, checkable rules over vague statements. This is what future code should conform to.
