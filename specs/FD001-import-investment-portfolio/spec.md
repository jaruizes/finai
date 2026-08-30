# Feature Specification: Import Investment Portfolio

**Feature ID**: FD001-import-investment-portfolio

**Feature Branch**: `FD001-import-investment-portfolio`

**Created**: 2026-08-30

**Status**: Draft — clarified 2026-08-30

**Source of intent**: [`feature-definitions/FD001-import-investment-portfolio.md`](../../feature-definitions/FD001-import-investment-portfolio.md) (human-authored Feature Definition — authoritative for product and domain intent)

**Input**: User description: "Create the specification from @feature-definitions/FD001-import-investment-portfolio.md. Treat this human-authored Feature Definition as the authoritative source of product and domain intent. Do not introduce new domain requirements. Do not resolve ambiguities by assumption. If something required for the specification is unresolved, mark it for clarification. Preserve the feature identifier and name exactly: FD001-import-investment-portfolio."

## Overview

FinAI must know what an investor currently holds before any later capability can reason about it.
This feature is the **initial import** of one investment portfolio into FinAI: the investor supplies
a portfolio name and its current positions, FinAI validates the entire submission, and — only if
every part is valid — establishes the Portfolio, its Positions, the Financial Instruments those
Positions represent, and any investor-supplied Investment Theses as canonical domain information.

The import records the investor's **stated** portfolio as a current-state snapshot. It does not
enrich, value, analyse, or make recommendations about the portfolio, and it does not represent the
transactions or acquisition lots that produced the current state. An invalid import establishes
nothing.

This specification formalises the Feature Definition and introduces no new domain requirements.
Points the Feature Definition left unresolved were decided by the product owner and are recorded
under **Clarifications**; a small number of purely representational details are deferred to the
interface contract (see **Dependencies**).

## Clarifications

### Session 2026-08-30

- Q: Does an import record an explicit snapshot "as-of" time and/or a source descriptor? → A: FinAI
  generates and stores an `importedAt` timestamp at import time. The investor does not supply an
  as-of timestamp, and no separate source descriptor is captured in this feature.
- Q: Are Ticker and Exchange MIC compared exactly as supplied, or normalised first? → A: Trim
  surrounding whitespace and upper-case both Ticker and Exchange MIC before identity comparison and
  duplicate detection; the canonical Financial Instrument stores the normalised values.
- Q: Must each validation error identify the offending element in machine-readable form? → A: Yes.
  Every validation error includes a machine-readable location identifying the offending field or
  element (for example `positions[2].quantity`).
- Q: Are non-integer (fractional) quantities permitted for STOCK/ETF positions? → A: Yes. Any
  quantity greater than zero is allowed, including fractional quantities.
- Q: When supplied, what values are valid for Average Acquisition Price? → A: It remains optional;
  when supplied it must be numeric and greater than zero (zero, negative, or non-numeric values are
  validation errors).
- Q: How are empty/whitespace Portfolio Name and Investment Thesis text handled, and are there
  length limits? → A: Portfolio Name is trimmed and must contain at least one non-whitespace
  character. A missing, empty, or whitespace-only Investment Thesis means no thesis is present. No
  maximum text length is introduced in this feature.
- Q: If an import references an already-established canonical Financial Instrument with a different
  (or no) Display Name, which Display Name applies? → A: Importing a portfolio must not update the
  Display Name (or any attribute) of an already-established canonical Financial Instrument. No
  reconciliation or update behaviour for canonical instruments is introduced in this feature.
- Q: How is the owning investor established for an import? → A: Investor identity is supplied by the
  calling/authentication context and is not part of the import payload. The established Portfolio is
  associated with that investor.
- Q: Must the validation errors in a failed outcome be returned in a deterministic order? → A: Yes.
  Portfolio-level errors first, then position-level errors ordered by position occurrence in the
  submission, then by field/error within each position.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Establish a portfolio from a valid import (Priority: P1)

An investor submits one portfolio: a name they choose, and one or more current positions. Each
position names a supported financial instrument (a stock or ETF, identified by ticker and exchange),
a quantity greater than zero, and the currency of its acquisition information; optionally an average
acquisition price and an investment thesis. FinAI validates the whole portfolio and establishes it
as canonical domain information, returning a success code and the generated Portfolio ID.

**Why this priority**: Establishing a portfolio in the domain model is the entire purpose of the
feature and the precondition for everything else in FinAI. A single successful import is the minimum
viable outcome.

**Independent Test**: Submit a portfolio with a name and one valid position (supported instrument,
quantity > 0, valid ISO 4217 acquisition currency) and confirm the outcome is success, a Portfolio
ID is returned, and the Portfolio, its Position, and its Financial Instrument are afterwards
retrievable as canonical domain information.

**Acceptance Scenarios**:

1. **Given** a portfolio named "Long-Term Growth" with two positions — one STOCK and one ETF, each
   with a ticker, an ISO 10383 exchange MIC, a quantity greater than zero, and an ISO 4217
   acquisition currency — **When** the investor imports it, **Then** the outcome contains a success
   code and a generated Portfolio ID, both positions and their instruments are established, and the
   Portfolio records an `importedAt` timestamp and the investor from the calling context.
2. **Given** a valid position for which the investor does not provide an Average Acquisition Price
   but does provide the Acquisition Currency, **When** the portfolio is imported, **Then** the
   position is accepted and established.
3. **Given** a position with a fractional quantity greater than zero, **When** the portfolio is
   imported, **Then** the position is accepted with that fractional quantity.
4. **Given** a position that includes free-form Investment Thesis text, **When** the portfolio is
   imported, **Then** the thesis text is preserved exactly and associated with that position.
5. **Given** a position with no Investment Thesis (or empty / whitespace-only thesis text), **When**
   the portfolio is imported, **Then** the position is still valid and established with no thesis.
6. **Given** a ticker and exchange MIC imported earlier in a different portfolio (in any letter case
   or with surrounding whitespace), **When** a new portfolio referencing the same ticker and MIC is
   imported, **Then** the position resolves to the same canonical Financial Instrument, and that
   instrument's existing Display Name is not changed by the new import.

---

### User Story 2 - Reject an invalid import with all errors reported together (Priority: P2)

An investor submits a portfolio that contains one or more mistakes. FinAI examines the entire
portfolio, rejects the import as a whole, establishes nothing, and returns every detected validation
error together — each with a stable `PORT-IMP-NNNN` code, a human-readable message, and a
machine-readable location — in a deterministic order.

**Why this priority**: Partial or silent acceptance of a flawed portfolio would corrupt the domain
model and every downstream capability. Reporting all errors at once (not stopping at the first) lets
the investor fix the submission in one pass. It is only meaningful once import (US1) exists.

**Independent Test**: Submit a portfolio containing several distinct errors across different
positions and confirm the outcome is failure, that all of those errors are returned together with
codes, messages, and locations in the specified order, and that no Portfolio, Position, Financial
Instrument, or Thesis from that submission exists afterwards.

**Acceptance Scenarios**:

1. **Given** a position whose quantity is zero, negative, or non-numeric, **When** the portfolio is
   imported, **Then** the import fails, an error identifies the invalid quantity with its location,
   and nothing is established.
2. **Given** a position whose Average Acquisition Price is supplied as zero, negative, or
   non-numeric, **When** the portfolio is imported, **Then** the import fails with an error located
   at that position's average acquisition price.
3. **Given** two positions in the portfolio that reference the same ticker and exchange MIC (after
   trimming and upper-casing), **When** the portfolio is imported, **Then** the import fails with a
   duplicate-instrument error.
4. **Given** a position whose instrument type is neither STOCK nor ETF, **When** the portfolio is
   imported, **Then** the import fails with an unsupported-instrument-type error.
5. **Given** a position whose Acquisition Currency is not a recognised ISO 4217 alphabetic code,
   **When** the portfolio is imported, **Then** the import fails with an invalid-currency error.
6. **Given** a portfolio whose name is empty or whitespace-only, **When** it is imported, **Then**
   the import fails with an error located at the portfolio name.
7. **Given** a portfolio with no positions, **When** it is imported, **Then** the import fails
   because a portfolio must contain at least one position.
8. **Given** an import that describes more than one portfolio, **When** it is submitted, **Then**
   the import fails because one import represents exactly one portfolio.
9. **Given** a portfolio in which the portfolio name is invalid and several positions carry
   different errors, **When** it is imported, **Then** the failed outcome lists every error
   together — portfolio-level errors first, then position-level errors by position occurrence and
   then by field — each with a `PORT-IMP-NNNN` code, a descriptive message, and a location, and
   nothing is established.

---

### User Story 3 - Capture the investor's rationale and partial acquisition information (Priority: P3)

For each position the investor may add free-form investment thesis text explaining why they hold it,
and may provide the average acquisition price when they know it (or omit it when they do not, while
still stating the acquisition currency). FinAI stores this information verbatim against the position.

**Why this priority**: Thesis text and acquisition price make the holding analysable by later
capabilities, but the portfolio is already usable without them. This is incremental value on top of
US1.

**Independent Test**: Import a position with a multi-line thesis and no average acquisition price;
confirm the thesis is retrievable unchanged and the position is valid with only the acquisition
currency supplied.

**Acceptance Scenarios**:

1. **Given** a position with an Average Acquisition Price greater than zero and an Acquisition
   Currency, **When** the portfolio is imported, **Then** both values are stored against the
   position exactly as supplied.
2. **Given** a position with an Investment Thesis of several paragraphs, **When** the portfolio is
   imported, **Then** the full text is preserved and attributed to the position, unaltered.
3. **Given** a position with an Investment Thesis, **When** it is established, **Then** FinAI has not
   generated, corrected, summarised, evaluated, or enriched that text in any way.

---

### Edge Cases

- **Portfolio with zero positions** → rejected (a portfolio must contain at least one position).
- **Import describing more than one portfolio** → rejected (one import = one portfolio).
- **Portfolio Name empty or whitespace-only** → rejected (must contain at least one non-whitespace
  character after trimming).
- **Quantity of zero, negative, or non-numeric** → rejected. **Fractional quantity greater than
  zero** → accepted. (Short positions are out of scope.)
- **Average Acquisition Price supplied as zero, negative, or non-numeric** → rejected. **Average
  Acquisition Price omitted** → position remains valid.
- **Ticker or Exchange MIC differing only by letter case or surrounding whitespace** → treated as
  the same value after trimming and upper-casing; two such positions in one portfolio are a
  duplicate Financial Instrument and are rejected.
- **Ticker or Exchange MIC that is whitespace-only** → treated as missing → rejected.
- **Instrument type other than STOCK or ETF** → rejected.
- **Acquisition Currency missing** → rejected (required even when the price is unknown).
- **Acquisition Currency not a valid ISO 4217 alphabetic code** → rejected.
- **Exchange not a valid ISO 10383 MIC**, or **ticker / exchange / instrument type missing** →
  rejected.
- **Investment Thesis missing, empty, or whitespace-only** → position is valid with no thesis.
- **Import references an already-established canonical Financial Instrument with a different or
  absent Display Name** → the position resolves to that instrument; its Display Name is not changed.
- **Multiple distinct errors, including a mix of portfolio-level and position-level errors** → all
  reported together in the deterministic order; nothing established.

## Requirements *(mandatory)*

Each requirement traces to the Feature Definition ("FD BR-n" = Business Rule n; other references name
the FD section) or to a **Clarifications** decision ("CL-n" = clarification item n, in the order
listed above). No requirement below extends the Feature Definition's domain scope.

### Functional Requirements

#### Import structure and portfolio

- **FR-001**: The system MUST accept an import that describes exactly one Portfolio and MUST reject
  an import that describes zero or more than one Portfolio. *(FD BR-1)*
- **FR-002**: The system MUST require a Portfolio Name — a human-readable value freely chosen by the
  investor. The system MUST trim surrounding whitespace from the Name, MUST reject a Name that has
  no non-whitespace character, MUST NOT otherwise transform it, MUST NOT require it to be unique,
  and MUST NOT impose a maximum length. *(FD BR-3; Portfolio → Required information; CL-6)*
- **FR-003**: The system MUST reject a Portfolio that contains no Position. *(FD BR-2; Portfolio
  Definition)*
- **FR-004**: The system MUST generate a stable Portfolio ID for each established Portfolio,
  assigned by FinAI and independent of the Portfolio Name. *(FD BR-4; Portfolio → Generated by
  FinAI)*
- **FR-005**: The system MUST generate and store an `importedAt` timestamp on each established
  Portfolio, recording when FinAI imported it. The investor MUST NOT supply an as-of timestamp, and
  the system MUST NOT capture a separate source descriptor in this feature. *(CL-1)*
- **FR-006**: The system MUST associate each established Portfolio with the investor identity
  supplied by the calling/authentication context. Investor identity MUST NOT be part of the import
  payload. *(FD Portfolio → Definition; CL-8)*
- **FR-007**: The system MUST NOT require, derive, or store a portfolio valuation currency, an
  account or broker association, transaction history, orders, or cash movements as part of a
  Portfolio. *(FD Portfolio → Does not represent)*

#### Positions

- **FR-008**: The system MUST associate every Position in the import with the single imported
  Portfolio; a Position belongs to exactly one Portfolio. *(FD BR-5; Position → Relationships)*
- **FR-009**: The system MUST require each Position to reference exactly one Financial Instrument.
  *(FD BR-6; Position → Relationships)*
- **FR-010**: The system MUST require each Position's Quantity to be numeric and greater than zero,
  MUST accept fractional quantities greater than zero, and MUST reject a Quantity that is zero,
  negative, non-numeric, or absent. *(FD BR-11; Position → Business constraints; CL-4)*
- **FR-011**: The system MUST require each Position to specify an Acquisition Currency as an ISO
  4217 alphabetic currency code, including when no Average Acquisition Price is provided, and MUST
  reject a Position whose Acquisition Currency is missing or not a recognised ISO 4217 alphabetic
  code. *(FD BR-13; Position → Business constraints; Scenario "Invalid currency")*
- **FR-012**: The system MUST accept a Position that has no Average Acquisition Price. *(FD BR-12;
  Scenario "Unknown acquisition price")*
- **FR-013**: The system MUST accept and store an optional Average Acquisition Price when the
  investor provides it, and MUST reject a supplied Average Acquisition Price that is non-numeric,
  zero, or negative. *(FD Position → Optional information; CL-5)*
- **FR-014**: The system MUST treat a Position as the aggregated current holding of one Financial
  Instrument and MUST NOT require or represent individual purchases or sales, orders, tax lots,
  transaction history, current market value, or unrealised profit or loss. *(FD BR-10; Position →
  Does not represent)*
- **FR-015**: The system MUST generate a stable Position ID for each established Position. *(FD
  Position → Generated by FinAI)*
- **FR-016**: The system MUST reject an import in which the same Financial Instrument (determined by
  normalised identity, FR-021) occurs in more than one Position of the Portfolio. *(FD BR-7;
  Scenario "Duplicate financial instrument")*
- **FR-017**: The system MUST allow a Position to have zero or one Investment Thesis. *(FD Investment
  Thesis → Relationships)*

#### Financial Instruments

- **FR-018**: The system MUST support Financial Instruments of type STOCK or ETF only, and MUST
  reject a Position whose Instrument Type is anything else. *(FD BR-14; Scenario "Unsupported
  financial instrument")*
- **FR-019**: The system MUST require each Financial Instrument to specify a Ticker, an Exchange,
  and an Instrument Type, and MUST reject a Financial Instrument missing any of these (a value that
  is whitespace-only after trimming counts as missing). *(FD Financial Instrument → Required
  information; CL-2)*
- **FR-020**: The system MUST require the Exchange to be expressed as an ISO 10383 Market Identifier
  Code (MIC) and MUST reject an Exchange value that is not a recognised MIC. *(FD BR-9)*
- **FR-021**: The system MUST trim surrounding whitespace from, and upper-case, both the Ticker and
  the Exchange MIC before determining Financial Instrument identity, detecting duplicates (FR-016),
  and resolving instruments across Portfolios (FR-023). The canonical Financial Instrument MUST
  store the normalised Ticker and Exchange MIC. *(CL-2)*
- **FR-022**: The system MUST determine Financial Instrument identity in this version solely by the
  combination of normalised Ticker and normalised Exchange MIC, and MUST NOT accept a Ticker alone
  as sufficient identification. *(FD BR-8; Financial Instrument → Definition; CL-2)*
- **FR-023**: The system MUST assign a canonical Financial Instrument ID to each distinct normalised
  Ticker + Exchange MIC, and MUST resolve Positions in different Portfolios that reference the same
  normalised Ticker + Exchange MIC to the same canonical Financial Instrument. *(FD Financial
  Instrument → Generated by FinAI, Relationships)*
- **FR-024**: The system MUST accept an optional Financial Instrument Display Name and record it
  when the canonical Financial Instrument is first established. *(FD Financial Instrument → Optional
  information)*
- **FR-025**: When a Position references an already-established canonical Financial Instrument, the
  import MUST NOT modify that instrument's Display Name or any other attribute. This feature
  introduces no reconciliation or update behaviour for canonical Financial Instruments. *(CL-7)*
- **FR-026**: The system MUST NOT establish ISIN or other additional identifiers, market price,
  company fundamentals, sector/industry/geographical classification, market events, risk
  information, or any external enrichment data for a Financial Instrument. *(FD Financial Instrument
  → Does not represent; FD BR-20; Out of Scope)*

#### Investment Thesis

- **FR-027**: The system MUST accept an optional free-form Investment Thesis text supplied by the
  investor for a Position, MUST preserve non-empty thesis text exactly as supplied, and MUST
  associate it with that single Position. A missing, empty, or whitespace-only Investment Thesis
  MUST be treated as no thesis present. The system MUST NOT impose a maximum thesis length. *(FD
  BR-15; Investment Thesis → Definition, Relationships; Scenario "Portfolio with investment thesis";
  CL-6)*
- **FR-028**: The system MUST NOT generate, correct, enrich, infer, summarise, evaluate, or
  otherwise alter Investment Thesis content, and MUST NOT treat it as advice, a recommendation, a
  prediction, or a buy/sell/hold instruction. *(FD BR-16; Investment Thesis → Does not represent;
  Out of Scope)*

#### Validation and processing boundary

- **FR-029**: The system MUST NOT generate, correct, enrich, or infer any financial information
  supplied during import, including via a language model. *(FD BR-16)*
- **FR-030**: The system MUST validate the complete supplied Portfolio before establishing any
  canonical portfolio information. *(FD BR-17)*
- **FR-031**: The system MUST examine the complete supplied Portfolio and return all detected
  validation errors together in a single failed outcome, rather than stopping at the first error.
  *(FD BR-18; Scenario "Multiple validation errors")*
- **FR-032**: The system MUST ensure that a failed import establishes no Portfolio, Position,
  Financial Instrument, or Investment Thesis — there is no partially established portfolio. *(FD
  BR-19; FD Scope; Import Outcomes → Failed Import)*
- **FR-033**: An import MUST NOT trigger market-data retrieval, portfolio valuation, risk analysis,
  investment analysis, or recommendations. *(FD BR-20)*

#### Outcomes

- **FR-034**: On a fully valid import, the system MUST establish the Portfolio, its Positions, their
  Financial Instruments, and any Investment Theses as canonical domain information available to
  later FinAI capabilities. *(FD Import Outcomes → Successful Import)*
- **FR-035**: A successful outcome MUST contain a stable machine-readable success code and the
  generated Portfolio ID. The success code is NOT required to follow a namespaced convention
  analogous to `PORT-IMP-NNNN`. *(FD Import Outcomes → Successful Import; Resolved Design
  Decisions)*
- **FR-036**: A failed outcome MUST contain every detected validation error; each error MUST carry
  a stable machine-readable error code in the `PORT-IMP-NNNN` format, a human-readable descriptive
  message explaining the reason, and a machine-readable location identifying the offending field or
  element (for example `positions[2].quantity`; portfolio-level errors locate the portfolio-level
  field). *(FD Import Outcomes → Failed Import, Error Identification; Resolved Design Decisions;
  CL-3)*
- **FR-037**: The system MUST return the validation errors in a deterministic order: portfolio-level
  errors first, then position-level errors ordered by the position's occurrence in the submission,
  then by field/error within each position. *(CL-9)*
- **FR-038**: Error codes MUST be stable and MUST NOT depend on the language used for the
  descriptive message; descriptive messages MAY be localised in future without changing the code.
  *(FD Error Identification)*
- **FR-039**: The exact catalogue and meaning of individual `PORT-IMP-NNNN` error codes, and the
  exact grammar of the error location expression, MUST be defined as part of the formal interface
  contract during planning, not in this specification. *(FD Error Identification)*

### Key Entities *(include if feature involves data)*

- **Portfolio**: a collection of an investor's current investment positions; the context in which
  Positions exist. Required: Name (trimmed, at least one non-whitespace character), at least one
  Position. Associated with: the owning investor identity from the calling context. Generated by
  FinAI: Portfolio ID, `importedAt` timestamp. Does not carry a valuation currency, account, or
  transaction history.
- **Position**: the investor's current aggregated holding of one Financial Instrument within one
  Portfolio. Required: Financial Instrument, Quantity (numeric, > 0, fractional allowed),
  Acquisition Currency (ISO 4217 alphabetic). Optional: Average Acquisition Price (numeric, > 0 when
  present), Investment Thesis. Generated by FinAI: Position ID.
- **Financial Instrument**: an identifiable stock or ETF held through a Position. Required: Ticker,
  Exchange (ISO 10383 MIC), Instrument Type (STOCK | ETF). Optional: Display Name (set once, at
  first establishment; never updated by a later import). Generated by FinAI: canonical Financial
  Instrument ID. Identity = normalised (trimmed, upper-cased) Ticker + Exchange MIC, stored in
  normalised form. Shared across Portfolios; appears at most once per Portfolio.
- **Investment Thesis**: the investor's own free-form rationale for holding a Position. Zero or one
  per Position; belongs to exactly one Position. Present only when non-empty, non-whitespace text is
  supplied. Investor-authored only; never FinAI-generated.
- **Import Outcome**: the result of one import attempt — either Success (success code + Portfolio
  ID) or Failure (an ordered collection of Validation Errors).
- **Validation Error**: one detected reason an import is invalid. Carries a stable `PORT-IMP-NNNN`
  code, a human-readable descriptive message, and a machine-readable location for the offending
  field or element.
- **Reference standards** (external, not owned here): ISO 4217 alphabetic currency codes; ISO 10383
  Market Identifier Codes.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 100% of imports in which the entire Portfolio satisfies every stated rule are
  established as canonical domain information and return a success code plus a Portfolio ID, with an
  `importedAt` timestamp recorded on the Portfolio.
- **SC-002**: 100% of imports containing at least one rule violation are rejected and leave zero
  Portfolio, Position, Financial Instrument, and Investment Thesis data established.
- **SC-003**: For an invalid import containing N distinct validation errors, all N errors are
  returned together in the single failed outcome (0% stop-at-first-error).
- **SC-004**: 100% of returned validation errors carry a `PORT-IMP-NNNN`-format code, a non-empty
  human-readable message, and a machine-readable location.
- **SC-005**: Re-validating the identical invalid import yields the identical ordered list of error
  codes and locations on every run.
- **SC-006**: 100% of Investment Thesis texts supplied (non-empty) are retrievable after import
  identical, character for character, to the text supplied.
- **SC-007**: 100% of imported values are retrievable identical to the supplied values, except for
  the explicitly specified normalisations — Portfolio Name trimmed; Ticker and Exchange MIC trimmed
  and upper-cased — with no other correction, substitution, inference, or enrichment by FinAI.
- **SC-008**: A Position supplied without an Average Acquisition Price is accepted in 100% of cases
  where its other required information (including a valid Acquisition Currency) is valid.
- **SC-009**: Importing the same Ticker + Exchange MIC across separate Portfolios — in any letter
  case or with surrounding whitespace — resolves to the same canonical Financial Instrument ID in
  100% of cases, and never alters the instrument's Display Name.
- **SC-010**: 0 market-data retrievals, valuations, risk analyses, investment analyses, or
  recommendation actions are triggered by any import.

## Assumptions

- One import is a single operation carrying exactly one Portfolio (FD BR-1).
- "Establish as canonical domain information" means the data becomes available to later FinAI
  capabilities; the storage and access mechanism is an implementation concern decided during
  planning.
- Authoritative ISO 4217 (currency) and ISO 10383 (MIC) code lists are available to validate
  against; maintaining those lists is outside this feature.
- The calling/authentication context reliably provides the owning investor identity for every
  import (CL-8).
- Only the first-version constraints stated in the Feature Definition apply (STOCK and ETF only;
  Ticker + Exchange MIC as the sole identification mechanism).

## Out of Scope

Per the Feature Definition, the following are explicitly outside this feature:

- Portfolio re-import and reconciliation; portfolio, position, and transaction history and
  versioning.
- Individual purchases or sales, orders, tax lots, cash movements.
- Short positions.
- Portfolio valuation, valuation currency, profit and loss, weighting, performance, currency
  conversion.
- Market prices and market-data retrieval.
- Instrument enrichment, additional identifiers (e.g. ISIN), company fundamentals, sector/industry/
  geographical enrichment; reconciliation or update of canonical Financial Instrument attributes.
- Position-role or investment-purpose classification.
- Market news and market events.
- Risk calculation, portfolio analysis, investment-thesis evaluation.
- AI-generated investment theses, investment recommendations, trade execution.
- The internationalisation mechanism for error messages.

## Dependencies

- The FinAI **canonical domain / semantic model**, to which this feature contributes the Portfolio,
  Position, and Financial Instrument concepts.
- Authoritative **ISO 4217** currency code and **ISO 10383** Market Identifier Code references for
  validation.
- The **calling/authentication context** that supplies the owning investor identity.
- The **formal import interface contract** (operation shape, success/error payloads, the full
  `PORT-IMP-NNNN` error catalogue, the exact error-location grammar, and the representation of the
  `importedAt` timestamp), to be defined during `/speckit-plan`.
