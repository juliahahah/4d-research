# knowledge.md template

A factual survey of the project, built by reading — not guessing. Every statement should be traceable to a file you actually read. Mark anything unverified clearly.

Use this structure:

```markdown
# Project Knowledge

_Last surveyed: <date>. Based on reading the files listed under "Sources"._

## Overview
One paragraph: what this project is and does, based on README and entry points.

## Tech stack
- Languages, frameworks, runtimes (with versions from manifest files)
- Build / package tooling
- Test framework(s)
- Notable dependencies that shape the architecture

## Structure
Annotated directory tree of the meaningful parts (skip node_modules, build output, etc.).
For each significant folder, one line on its responsibility.

## Architecture & data flow
How the pieces fit: layers, entry points, how a request/action flows through the system.
Reference concrete files/modules.

## Key modules relevant to typical work
For the areas most likely to be touched, note the file, its responsibility, and important types/functions.

## Common commands
How to build, run, test, and lint the project — the actual commands, taken from README / package.json scripts / Makefile / CI config (not guessed). e.g. install, dev server, run tests, run a single test, lint/format.

## Conventions observed
Anything structural worth knowing (folder-per-feature, config location, env handling).
(Coding style lives in style.md — keep it there.)

## Uncertainties / not yet read
Parts of the project that are relevant but were not read in depth. Be honest here so future plans know what's unverified.

## Sources
List the files actually read to produce this document.
```

Keep it factual and skimmable. This document exists so future design work doesn't re-survey from scratch.
