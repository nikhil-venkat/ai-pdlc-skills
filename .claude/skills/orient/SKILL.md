---
name: orient
description: Read-only orientation of an unfamiliar codebase. Produces a compact map (stack and commands, entry points, one traced request flow, background jobs, data model, tests, conventions) and names the files most relevant to a given feature. Use before /spec or /build in code you have not worked in, at the start of a live coding session, or when the user says "orient", "get oriented", or "map this repo". Pass the feature request as args if known.
---

# Orient (Codebase Orientation)

> **Role in the workflow: step 0, optional.** See [docs/SPECS.md](../../../docs/SPECS.md).
> - **Runs before** `/spec`, whenever the codebase is unfamiliar (new repo, new area, interview exercise).
> - **Input:** the repo, and optionally the feature request.
> - **Output:** a one-screen map of the repo and the files and existing patterns relevant to the feature. Nothing is changed.

## Overview

Understand before you change. Most bad AI-generated code in an unfamiliar repo comes from not knowing its conventions: invented helpers, a second way of doing errors, a test style nobody else uses. This skill builds the map that `/spec` and `/build` then hold the code to.

**Read-only.** Do not edit, create, or delete files. Do not install dependencies, run migrations, or start services. Reading files, listing directories and printing help targets (`make help`, `npm run`) are fine.

**Be fast.** Aim for a usable map in a few minutes, not an audit. Breadth first, then depth only on the path that matters for the feature.

## When to Use

- First time in a repo, or in an area of a repo you have not touched
- Before writing a spec or a plan for a feature in unfamiliar code
- Live coding exercises in a codebase you are handed
- When another agent's output keeps missing the repo's conventions

## Steps

1. **Top level.** Read the README, build files (`package.json`, `Makefile`, `pyproject.toml`, `go.mod`, `build.gradle`), `docker-compose`, CI config, and any `CLAUDE.md`, `AGENTS.md` or `docs/`. Note the language, frameworks, and the exact commands to install, run, test and verify.
2. **Entry points.** Find where processes start: HTTP server and route registration, background workers or job consumers, CLIs, scheduled jobs.
3. **One request flow.** Trace a single real path end to end, for example route → handler → service → data access → database, or event → consumer → side effect. Name the actual files and functions.
4. **Background work.** If the repo uses queues, workflow engines, or schedulers, list each job or workflow, what triggers it, its retry and timeout settings, and how duplicates are prevented.
5. **Data model.** Read migrations, schema files or ORM models, plus seed data. List the tables or collections, the columns that matter, constraints (unique keys, foreign keys, status enums), and the domain vocabulary.
6. **Tests.** Where they live, the framework, how external dependencies are faked or mocked, and the exact command to run them. Note anything skipped or already failing.
7. **Conventions.** Error handling, validation, logging, naming, config and env vars, data access pattern. These are what every later change gets held to.
8. **If a feature was given.** Name the 3 to 5 files most relevant to it and why, and any existing code that already does something similar. That is the pattern to copy.

## Output Format

One screen, these headings, terse bullets, real file paths:

- **Stack and commands** (install / run / test / verify)
- **How a request flows** (one traced path, with arrows)
- **Background work** (only if present)
- **Data model** (tables and the columns that matter)
- **Tests** (location, command, mocking pattern)
- **Conventions to match**
- **Relevant to the feature** (files, plus the existing pattern to copy)
- **Open questions** (2 to 3 genuine ambiguities worth asking about before building)

End with a one-sentence summary of the repo that could be said out loud.

## Red Flags

- Editing anything during orientation
- Describing the architecture from file names alone, without reading the code that runs
- A map with no file paths
- Skipping the tests section; it is how every later step will verify
- Reading the whole repo when a feature was given; go deep only on its path
