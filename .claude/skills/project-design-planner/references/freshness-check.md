# Freshness check — reuse or refresh a prior survey?

`knowledge.md` and `style.md` are built once and reused. Before reusing them, do a lightweight check for whether the project has meaningfully changed since they were written. The goal is to avoid an expensive full re-survey while not planning against stale facts.

## Quick signals to check

1. **Dependency/manifest drift.** Compare current `package.json` / `pyproject.toml` / `requirements.txt` / lockfiles against what `knowledge.md` recorded. New frameworks or major-version bumps → refresh the affected sections.
2. **Structure drift.** Glance at the current top-level directory tree vs. the one in `knowledge.md`. New top-level modules or a reorganized layout → update the Structure and Architecture sections.
3. **Config drift.** Check whether linter/formatter configs changed since `style.md` was written. If so, refresh `style.md`.
4. **Relevance to this request.** Even if the project is broadly unchanged, the *specific area* this new requirement touches may not have been read in depth before (check the "Uncertainties / not yet read" section of `knowledge.md`). If so, deep-read that area now and extend `knowledge.md`.

If you can determine file modification times or a VCS diff cheaply, use them — but manifest/structure/config comparison is usually enough.

## Decision

- **No meaningful drift and the relevant area was already surveyed** → reuse both documents as-is. Note in `design.md` that the plan is based on the existing survey.
- **Partial drift** → update only the affected sections, bump the "Last surveyed" date, and note what changed.
- **Major drift** (stack change, large restructure) → re-survey from scratch.

Never silently plan against facts you suspect are stale. When in doubt, verify by reading.
