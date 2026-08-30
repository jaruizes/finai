# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working
with code in this repository.

## What FinAI is

FinAI is an experimental, AI-native portfolio intelligence platform:
deterministic financial analysis combined with a shared semantic model
and AI agents, to help investors understand portfolios, risk, investment
theses, and market events. It analyzes and explains; it does not give
personalized advice and does not execute transactions.

No application stack has been chosen yet. The repository currently holds
only Spec-Driven Development (SDD) scaffolding under `.specify/` and the
`speckit-*` skills. Do not add build, lint, or test commands to this
file until a feature plan selects the toolchain --- then document the
real commands here.

## Authority and precedence

The project constitution (`.specify/memory/constitution.md`) is the top
authority. Read it before non-trivial work. For any conflict, resolve in
this order (higher wins):

1.  **Constitution** --- principles and governance. Never work around
    it; change it only via `/speckit-constitution`.
2.  **Human feature definition**
    (`feature-definitions/FDNNN-short-name.md`) --- authoritative
    human-authored product and domain intent for the feature.
3.  **Feature specification** (`specs/FDNNN-short-name/spec.md`) ---
    formalized behavior and outcomes derived from the matching human
    feature definition.
4.  **Plan** (`specs/FDNNN-short-name/plan.md`) --- how it is built,
    including the Constitution Check.
5.  **Tasks** (`specs/FDNNN-short-name/tasks.md`) --- the ordered work
    derived from the plan.
6.  **CLAUDE.md** (this file) --- operational defaults when the above
    are silent.
7.  **Existing code** --- informative, not authoritative. Follow local
    conventions, but a pattern in the tree never justifies violating
    anything above it.

If a higher artifact is missing, wrong, or contradicts a lower one, stop
and flag it --- do not reconcile the gap silently in code.

## Spec-Driven Development workflow

All feature work goes through the Spec Kit pipeline. Each step is a
skill; do not skip ahead.

1.  `/speckit-specify` --- formalize an existing human-authored feature
    definition from `feature-definitions/FDNNN-short-name.md` into
    `specs/FDNNN-short-name/spec.md` and its feature branch. The
    generated specification directory name MUST exactly match the
    feature-definition filename without the `.md` extension. The spec
    covers outcomes and behavior, not implementation.
2.  `/speckit-clarify` --- resolve ambiguities in the spec before
    planning (recommended).
3.  `/speckit-plan` --- produce `plan.md` and design artifacts
    (`research.md`, `data-model.md`, `contracts/`, `quickstart.md`).
    This step runs the **Constitution Check**: unresolved principle
    violations must be justified in the plan's Complexity Tracking, or
    the design changes.
4.  `/speckit-tasks` --- generate dependency-ordered `tasks.md`.
5.  `/speckit-analyze` --- non-destructive consistency check across
    spec, plan, and tasks.
6.  `/speckit-implement` --- execute `tasks.md`.

`/speckit-checklist` and `/speckit-converge` help reconcile artifacts
against code. The full cycle with review gates is also defined in
`.specify/workflows/speckit/workflow.yml`.

Changes outside a feature (dependency bumps, CI, docs, tooling) still
respect the constitution and this file but do not need the full
pipeline.

### Feature naming and traceability

Every feature starts with a committed, human-authored definition under
`feature-definitions/`.

Feature definitions MUST use this naming convention:

`FDNNN-short-name.md`

where:

-   `FD` identifies the artifact as a human Feature Definition;
-   `NNN` is a zero-padded sequential number (`001`, `002`, `003`, ...);
-   `short-name` is a lowercase kebab-case description of the feature.

Example:

`feature-definitions/FD001-import-investment-portfolio.md`

The corresponding Spec Kit directory MUST use exactly the same basename,
without the `.md` extension:

`specs/FD001-import-investment-portfolio/`

Its specification is therefore:

`specs/FD001-import-investment-portfolio/spec.md`

Do not generate a different sequence number, rename the short name, or
use Spec Kit's default `NNN-short-name` directory naming when a Feature
Definition already exists. The Feature Definition name is the
traceability key for all artifacts belonging to that feature.

The feature branch SHOULD use the same feature identifier and short name
when the tooling permits it:

`FD001-import-investment-portfolio`

Current feature resolution still follows
`.specify/scripts/bash/common.sh`, in order: `SPECIFY_FEATURE_DIRECTORY`
env var → `.specify/feature.json` (machine-local, gitignored) → git
branch name.

Artifact templates live under `.specify/templates/*.md`. Helper scripts
(POSIX `sh`) live under `.specify/scripts/bash/`.

### Feature Specifications

Human-authored feature definitions under `feature-definitions/`
represent the authoritative product and domain intent for a feature. A
specification is a formalization of that definition, not a new source of
product requirements.

When generating or refining a specification from a feature definition:

-   Read the matching `feature-definitions/FDNNN-short-name.md` first.
-   Use the Feature Definition filename as the feature traceability
    identifier.
-   Generate the specification under `specs/FDNNN-short-name/`.
-   Do not invent domain requirements.
-   Do not resolve ambiguous business behavior by assumption.
-   Preserve the scope and intent of the feature definition.
-   Explicitly identify unresolved decisions for clarification.
-   Formalize requirements and acceptance criteria without introducing
    implementation or architecture decisions.
-   If Spec Kit defaults conflict with this naming convention, preserve
    the Feature Definition naming and flag any tooling change needed
    rather than silently using another name.

## Operating rules (summaries --- the constitution has the binding text)

-   **Domain ownership**: financial and portfolio concepts belong to the
    deterministic domain layer and the shared semantic model. Features
    and agents use them; they do not redefine them locally.
-   **Deterministic financial computation**: all portfolio math is
    deterministic and testable without a model call. Agents retrieve and
    explain; they never produce or override a reported number.
-   **Semantic-model discipline**: model domain concepts against the
    shared ontology / semantic layer and go through it rather than
    hard-coding source schemas. How data is retrieved is a per-feature
    decision, not a fixed one.
-   **Reproducibility**: record each input's source and as-of time; an
    analytical run must be reproducible from its recorded inputs.
-   **Evidence**: every analytical output cites the concrete evidence
    and provenance it rests on. When evidence is insufficient or stale,
    return an explicit "cannot determine", never a guess.
-   **Test-first**: for deterministic domain logic and published
    contracts, write the failing test before the implementation.

## Architecture restraint

-   Build the simplest thing that satisfies the current spec. No
    speculative abstraction layers, plugin systems, or configuration for
    needs that do not exist yet.
-   Default to a single in-process application. Add a separate service,
    datastore, queue, or any distributed component only when a current
    requirement forces it, and record that justification in the
    feature's `plan.md`.
-   Prefer choices that are cheap to reverse. Do not introduce a
    technology (framework, database, message bus, protocol, external
    provider) that no spec or plan has called for.

## Repository safety

-   No secrets in the repo or in code. `.env*` files are never committed
    (`.env.example` excepted).
-   Private and local datasets live only under gitignored paths
    (`data/private/`, `data/local/`). Never commit portfolio holdings or
    personal financial data.
-   Do not add --- or share code with --- any path that places, routes,
    or executes market transactions.
-   Do not send private portfolio data to third-party services unless a
    reviewed spec explicitly requires it.
-   `.claude/` and `CLAUDE.local.md` are gitignored and local-only. Do
    not commit generated artifacts, caches, or `.specify/feature.json`.
