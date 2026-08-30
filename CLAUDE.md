# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What FinAI is

FinAI is an experimental, AI-native portfolio intelligence platform: deterministic financial
analysis combined with a shared semantic model and AI agents, to help investors understand
portfolios, risk, investment theses, and market events. It analyzes and explains; it does not give
personalized advice and does not execute transactions.

No application stack has been chosen yet. The repository currently holds only Spec-Driven
Development (SDD) scaffolding under `.specify/` and the `speckit-*` skills. Do not add build, lint,
or test commands to this file until a feature plan selects the toolchain — then document the real
commands here.

## Authority and precedence

The project constitution (`.specify/memory/constitution.md`) is the top authority. Read it before
non-trivial work. For any conflict, resolve in this order (higher wins):

1. **Constitution** — principles and governance. Never work around it; change it only via
   `/speckit-constitution`.
2. **Feature specification** (`specs/NNN-*/spec.md`) — what the current feature must do.
3. **Plan** (`specs/NNN-*/plan.md`) — how it is built, including the Constitution Check.
4. **Tasks** (`specs/NNN-*/tasks.md`) — the ordered work derived from the plan.
5. **CLAUDE.md** (this file) — operational defaults when the above are silent.
6. **Existing code** — informative, not authoritative. Follow local conventions, but a pattern in
   the tree never justifies violating anything above it.

If a higher artifact is missing, wrong, or contradicts a lower one, stop and flag it — do not
reconcile the gap silently in code.

## Spec-Driven Development workflow

All feature work goes through the Spec Kit pipeline. Each step is a skill; do not skip ahead.

1. `/speckit-specify` — turn a feature description into `specs/NNN-short-name/spec.md` and its
   feature branch. The spec covers outcomes and behavior, not implementation.
2. `/speckit-clarify` — resolve ambiguities in the spec before planning (recommended).
3. `/speckit-plan` — produce `plan.md` and design artifacts (`research.md`, `data-model.md`,
   `contracts/`, `quickstart.md`). This step runs the **Constitution Check**: unresolved principle
   violations must be justified in the plan's Complexity Tracking, or the design changes.
4. `/speckit-tasks` — generate dependency-ordered `tasks.md`.
5. `/speckit-analyze` — non-destructive consistency check across spec, plan, and tasks.
6. `/speckit-implement` — execute `tasks.md`.

`/speckit-checklist` and `/speckit-converge` help reconcile artifacts against code. The full cycle
with review gates is also defined in `.specify/workflows/speckit/workflow.yml`.

Changes outside a feature (dependency bumps, CI, docs, tooling) still respect the constitution and
this file but do not need the full pipeline.

### Feature directory / branch conventions

- Feature dirs: `specs/NNN-short-name/` (sequential 3+ digit prefix; `.specify/init-options.json`).
- Current feature is resolved by `.specify/scripts/bash/common.sh`, in order:
  `SPECIFY_FEATURE_DIRECTORY` env var → `.specify/feature.json` (machine-local, gitignored) →
  git branch name.
- Artifact templates: `.specify/templates/*.md`. Helper scripts (POSIX `sh`):
  `.specify/scripts/bash/`.

## Operating rules (summaries — the constitution has the binding text)

- **Domain ownership**: financial and portfolio concepts belong to the deterministic domain layer
  and the shared semantic model. Features and agents use them; they do not redefine them locally.
- **Deterministic financial computation**: all portfolio math is deterministic and testable without
  a model call. Agents retrieve and explain; they never produce or override a reported number.
- **Semantic-model discipline**: model domain concepts against the shared ontology / semantic
  layer and go through it rather than hard-coding source schemas. How data is retrieved is a
  per-feature decision, not a fixed one.
- **Reproducibility**: record each input's source and as-of time; an analytical run must be
  reproducible from its recorded inputs.
- **Evidence**: every analytical output cites the concrete evidence and provenance it rests on.
  When evidence is insufficient or stale, return an explicit "cannot determine", never a guess.
- **Test-first**: for deterministic domain logic and published contracts, write the failing test
  before the implementation.

## Architecture restraint

- Build the simplest thing that satisfies the current spec. No speculative abstraction layers,
  plugin systems, or configuration for needs that do not exist yet.
- Default to a single in-process application. Add a separate service, datastore, queue, or any
  distributed component only when a current requirement forces it, and record that justification in
  the feature's `plan.md`.
- Prefer choices that are cheap to reverse. Do not introduce a technology (framework, database,
  message bus, protocol, external provider) that no spec or plan has called for.

## Repository safety

- No secrets in the repo or in code. `.env*` files are never committed (`.env.example` excepted).
- Private and local datasets live only under gitignored paths (`data/private/`, `data/local/`).
  Never commit portfolio holdings or personal financial data.
- Do not add — or share code with — any path that places, routes, or executes market transactions.
- Do not send private portfolio data to third-party services unless a reviewed spec explicitly
  requires it.
- `.claude/` and `CLAUDE.local.md` are gitignored and local-only. Do not commit generated
  artifacts, caches, or `.specify/feature.json`.
