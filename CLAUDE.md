# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this project is

FinAI is an experimental "AI-native portfolio intelligence platform." The intent (see `README.md`) is
to combine deterministic financial analysis, a financial ontology / knowledge graph, a semantic layer,
GraphRAG, and AI agents (exposed over MCP, event-driven) to help investors reason about portfolios,
risk, investment theses, and market news — with explainability, agent evaluation, and observability as
first-class concerns. It gives no financial advice and executes no transactions.

## Current state (read this first)

The repository is **greenfield**. There is no application code, no chosen language runtime, no build
or test tooling yet. What exists is the Spec-Driven Development (SDD) scaffolding under `.specify/` and
the `speckit-*` skills. The `.gitignore` anticipates a Python stack (`.venv`, `.ruff_cache`,
`.mypy_cache`, `.pytest_cache`) but nothing is committed to that yet.

Do not invent build/lint/test commands — there are none to document until the first feature is
implemented. When the stack is established, add its commands to this file.

## How work happens here: Spec-Driven Development

All feature work flows through spec-kit. The pipeline, each step backed by a skill:

1. `/speckit-constitution` — establish/amend project principles. **`.specify/memory/constitution.md`
   is still an unfilled template.** It must be populated before `plan` has real gates to check.
2. `/speckit-specify` — turn a feature description into `specs/NNN-short-name/spec.md` (creates the
   feature directory and, in a git repo, a `NNN-short-name` branch).
3. `/speckit-clarify` — resolve underspecified areas in the spec (optional but recommended before plan).
4. `/speckit-plan` — produce `plan.md` plus design artifacts (`research.md`, `data-model.md`,
   `contracts/`, `quickstart.md`) from the plan template.
5. `/speckit-tasks` — generate dependency-ordered `tasks.md`.
6. `/speckit-analyze` — non-destructive consistency check across spec/plan/tasks.
7. `/speckit-implement` — execute `tasks.md`.

`/speckit-converge` and `/speckit-checklist` assist with reconciling code against artifacts and with
custom review checklists. The full `specify → plan → tasks → implement` cycle with review gates is
also defined as a workflow in `.specify/workflows/speckit/workflow.yml`.

### Feature directory / branch conventions

- Feature dirs live at `specs/NNN-short-name/` (sequential 3+ digit prefix; see
  `.specify/init-options.json` → `feature_numbering: sequential`).
- The "current feature" is resolved by `.specify/scripts/bash/common.sh` from, in order: the
  `SPECIFY_FEATURE_DIRECTORY` env var, then `.specify/feature.json` (machine-local, gitignored,
  rewritten on every feature switch), then the git branch name.
- Templates that artifacts are generated from: `.specify/templates/*.md`.
- Helper scripts (bash, POSIX `sh`): `.specify/scripts/bash/` — `check-prerequisites.sh`,
  `create-new-feature.sh`, `setup-plan.sh`, `setup-tasks.sh`, `resolve-template.sh`.

## Conventions

- `.claude/` is gitignored — local skills and settings there are not shared. `CLAUDE.local.md` is
  gitignored too.
- Private/local data belongs in `data/private/` or `data/local/` (gitignored).
- Keep the "no financial advice / no trade execution" boundary intact in any feature design.
