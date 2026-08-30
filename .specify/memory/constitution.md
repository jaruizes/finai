<!--
Sync Impact Report
==================
Version change: (uninitialized template) → 1.0.0
Bump rationale: Initial ratification. All placeholder tokens replaced with concrete,
project-specific governance derived from README.md (project vision and "Core ideas").

Principles defined (8):
  - I. Deterministic Core, Probabilistic Edge (NON-NEGOTIABLE)
  - II. Explainability by Construction
  - III. Ontology-Grounded Knowledge
  - IV. Contract-First Interfaces
  - V. Evaluated & Observable
  - VI. Test-First for Deterministic Domain Logic & Contracts (NON-NEGOTIABLE)
  - VII. Advisory Boundary (NON-NEGOTIABLE)
  - VIII. Evolutionary Architecture & Simplicity

Sections:
  - Added: "Engineering Constraints & Safety" (was [SECTION_2_NAME])
  - Added: "Development Workflow & Quality Gates" (was [SECTION_3_NAME])
  - Added: "Governance" (fully specified)

Templates & guidance:
  - .specify/templates/plan-template.md — OK, resolves "Constitution Check" gates
    dynamically from this file; no edit required.
  - .specify/templates/spec-template.md — OK, no constitution coupling.
  - .specify/templates/tasks-template.md — OK, no constitution coupling.
  - CLAUDE.md — note that "constitution is still an unfilled template" is now stale;
    update on next unrelated edit (out of scope for this command).

Deferred / TODO items:
  - None. RATIFICATION_DATE set to today (first adoption of this constitution).
-->

# FinAI Constitution

FinAI is an experimental, AI-native portfolio intelligence platform. It explores how
deterministic financial analysis, semantic knowledge, and agentic AI combine to help investors
understand their portfolios, risks, investment theses, market events, and relevant news. This
constitution defines the core engineering principles that every feature, plan, and change MUST
satisfy; principles marked NON-NEGOTIABLE admit no exceptions.

## Core Principles

### I. Deterministic Core, Probabilistic Edge (NON-NEGOTIABLE)

Financial computation is the source of truth; language models are not.

- All portfolio math — valuations, returns, exposures, risk metrics, attributions — MUST be
  implemented as deterministic, pure functions that produce identical output for identical input
  and are independently unit-testable without a model call.
- LLM- and agent-based components MUST be confined to the "edge": retrieval, synthesis, natural-
  language explanation, summarization, and orchestration. They MUST NOT compute or override a
  number that the deterministic core is responsible for.
- Any figure presented to a user MUST originate from the deterministic core or a cited external
  data source, never from model free-generation.
- Analytical runs MUST be reproducible from their recorded inputs.

Rationale: Investors act on numbers. Reproducibility and auditability of those numbers is the
product's foundation; probabilistic components are additive value, not load-bearing for correctness.

### II. Explainability by Construction

Every analytical output MUST be traceable end to end, with evidence and provenance preserved.

- A result MUST be able to name its inputs, the data sources (with as-of timestamps) those inputs
  came from, and the computation or agent steps that produced it.
- Agent and analytical responses MUST cite the concrete evidence they draw on — documents, data
  records, graph nodes, events — each carrying provenance (source and as-of time). An answer that
  cannot be grounded MUST be surfaced as "insufficient evidence," not guessed.
- Evidence, provenance, and reproducibility are non-negotiable expectations of any feature that
  produces analytical output. Explainability is an acceptance criterion, not a follow-up task.

Rationale: Trust in an analytical tool depends on the user being able to check its reasoning;
opaque outputs are a liability in a financial context.

### III. Ontology-Grounded Knowledge

The financial ontology and semantic layer are the shared vocabulary.

- Domain concepts (instruments, issuers, sectors, factors, theses, events, positions) MUST be
  modeled against the shared ontology rather than redefined per feature.
- The semantic layer is the primary interface between raw data and consumers; features MUST NOT
  bypass it to hard-code source-specific schemas into business logic.
- Retrieval MAY use graph, vector, relational, time-series, or external-search backends — chosen to
  fit the nature of the information, not fixed in advance. GraphRAG is one option among several, not
  a requirement. Whatever backend is used, retrieved context MUST resolve to ontology concepts and
  carry provenance (Principle II).

Rationale: A single coherent semantic model is what lets deterministic analysis, retrieval, and
agents interoperate; the retrieval mechanism is an implementation choice that should follow the
data.

### IV. Contract-First Interfaces

Interfaces between independently owned components are declared and versioned before implementation.

- Public interfaces — agent tools, service APIs, shared schemas — MUST have an explicit contract
  (typed inputs, outputs, and error modes) specified before implementation begins.
- Contracts are versioned. A breaking change requires a new version, not a mutation of the existing
  one; consumers MUST tolerate unknown fields, and producers MUST NOT remove or repurpose existing
  fields within a version.
- MCP SHOULD be preferred for agent tools that are exposed externally or where cross-tool
  interoperability is valuable. Internal domain services are NOT required to be expressed as MCP
  tools.
- Event-driven communication is OPTIONAL. It MUST be introduced only when an explicit requirement
  (decoupling, asynchrony, fan-out, auditability) justifies it, recorded in the feature's plan —
  never as a default. When used, event schemas are versioned contracts under the rules above.

Rationale: Stable, declared seams keep the system maintainable and its parts independently
testable. The transport and protocol for those seams should follow real requirements rather than be
mandated up front.

### V. Evaluated & Observable

A component is not "done" until it can be measured in development and in production.

- Every agent, prompt, and non-deterministic pipeline MUST ship with an evaluation suite covering
  at minimum: task accuracy, faithfulness/groundedness, and regression cases for known failures.
- Changes to prompts, models, retrieval, or agent wiring MUST be gated on their evaluation results;
  a regression in a tracked metric blocks the change.
- All components MUST emit structured logs, traces spanning agent/tool/retrieval steps, and metrics.
  A request MUST be reconstructable from telemetry alone.

Rationale: Probabilistic systems drift silently; without evaluation and observability, quality
regressions are discovered by users rather than by the team.

### VI. Test-First for Deterministic Domain Logic & Contracts (NON-NEGOTIABLE)

- For deterministic financial/domain logic (Principle I) and for every published contract
  (Principle IV), tests MUST be written before the implementation and MUST fail first.
- Red-Green-Refactor applies to this scope: characterize the desired behavior as a failing test,
  implement until it passes, then refactor.
- Every published contract, and every event schema where events are used, MUST have contract tests.
- Compliance is judged by the presence, quality, and coverage of tests over the deterministic
  scope — not by commit ordering, commit messages, or repository history.
- Exploratory prompt/agent tuning is exempt and is governed by Principle V instead.

Rationale: The core's value is its correctness guarantee; test-first is the cheapest way to keep
that guarantee true as the system grows.

### VII. Advisory Boundary (NON-NEGOTIABLE)

FinAI analyzes and informs; it does not personally advise or act.

- FinAI MAY analyze risks, scenarios, exposures, investment theses, market events, and changes
  in assumptions, and MAY explain their implications for a portfolio.
- FinAI MUST NOT present generated output as personalized financial advice or as a recommendation
  to buy, sell, or hold.
- FinAI MUST NOT autonomously decide or execute buy/sell/hold actions, and MUST NOT contain a
  code path that places, routes, or executes transactions.
- Analytical output MUST be presented as information and analysis, with the experimental,
  non-advisory nature of the project clear.
- Private portfolio data MUST stay within designated local/private storage and MUST NOT be sent to
  third-party services except as an explicitly reviewed, documented feature requirement.

Rationale: The project's scope is understanding — risk, scenarios, theses, shifting assumptions —
not personalized advice or execution. Analyzing investment questions is in scope; speaking as an
adviser or acting as an agent in the market is not.

### VIII. Evolutionary Architecture & Simplicity

Architecture serves current requirements and is allowed to change as they do.

- Every architectural or infrastructure choice — additional services, message buses, extra
  datastores, orchestration layers, distribution — MUST be justified by a requirement that exists
  now, recorded in the feature's plan.
- Speculative generality and "we might need it later" complexity are rejected by default. Prefer the
  simplest design that satisfies the current spec.
- Start from an in-process, modular composition. Introduce process boundaries and distributed
  systems only when a concrete constraint (scaling, isolation, independent deployment, team
  boundaries) demands it.
- Favor decisions that are cheap to reverse over decisions that lock the system in.

Rationale: Premature distribution and speculative infrastructure are the most expensive mistakes to
unwind. Keeping architecture proportional to real requirements preserves the ability to evolve.

## Engineering Constraints & Safety

- **Architecture**: A knowledge graph plus semantic layer as the shared knowledge substrate;
  deterministic analytical libraries as independently testable units; the simplest composition that
  meets current requirements (Principle VIII). Transports and protocols (MCP, events, RPC) are
  chosen per requirement, not mandated.
- **Reproducibility**: External data pulls MUST record source and as-of time. Analytical runs MUST
  be reproducible from recorded inputs.
- **Secrets & data**: No secrets in the repository or in code. Private and local datasets live only
  under ignored paths (e.g. `data/private/`, `data/local/`). `.env` files are never committed.
- **Dependencies**: New runtime dependencies and any new external data provider MUST be justified in
  the feature's plan (`research.md`) against a lighter alternative.
- **Failure posture**: When evidence is insufficient or a data source is stale/unavailable, the
  system degrades to an explicit "cannot determine" state rather than a fabricated answer.

## Development Workflow & Quality Gates

- **Spec-Driven Development**: All feature work flows through the Spec Kit pipeline —
  `/speckit-specify` → `/speckit-clarify` → `/speckit-plan` → `/speckit-tasks` →
  `/speckit-analyze` → `/speckit-implement`. Code changes without a corresponding spec and plan are
  out of process.
- **Constitution Check**: `/speckit-plan` MUST evaluate the design against every principle above and
  record the result. Violations MUST be resolved or explicitly justified in the plan's Complexity
  Tracking section before `/speckit-tasks` runs.
- **Definition of Done** for any feature:
  1. Deterministic domain-logic and contract tests written first and passing (Principle VI).
  2. Evaluation suite present and green for any agent/prompt/pipeline touched (Principle V).
  3. Outputs are traceable and cite their evidence with provenance (Principle II).
  4. Structured logging, tracing, and metrics emitted (Principle V).
  5. Advisory Boundary review passed (Principle VII).
  6. Architecture and infrastructure additions justified by a current requirement (Principle VIII).
- **Review**: Every change MUST be reviewed against this constitution. A reviewer citing a principle
  violation blocks merge until resolved or formally justified.

## Governance

- This constitution supersedes other development practices and conventions where they conflict.
- **Amendments**: Proposed via a change to this file that includes an updated Sync Impact Report,
  the rationale, and the version bump. Amendments take effect when merged.
- **Versioning policy** (semantic):
  - **MAJOR**: Removal or backward-incompatible redefinition of a principle or governance rule.
  - **MINOR**: New principle or section added, or existing guidance materially expanded.
  - **PATCH**: Clarifications, wording, and non-semantic refinements.
- **Compliance review**: `/speckit-plan` and `/speckit-analyze` are the automated gates.
  Human review MUST confirm compliance before merge. Unjustified complexity or principle
  violations are grounds to reject a change.
- **Runtime guidance**: `CLAUDE.md` provides agent operating guidance for this repository and MUST
  stay consistent with this constitution; on conflict, this constitution wins.

**Version**: 1.0.0 | **Ratified**: 2026-08-30 | **Last Amended**: 2026-08-30
