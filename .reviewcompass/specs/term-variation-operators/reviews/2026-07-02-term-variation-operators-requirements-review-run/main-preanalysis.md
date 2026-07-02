# Main Preanalysis: term-variation-operators requirements review

## Source materials read and purpose

- `intent/INTENT.md`, `intent/NON_GOALS.md`, `intent/DESIGN_PRINCIPLES.md` (full text): project-level purpose, exclusions, staged-development principle.
- `stages/feature-partitioning/2026-07-02-proposal.md` (excerpt): approved scope and dependency (`term-variation-operators` depends on `term-representation`, hard).
- `.reviewcompass/specs/term-representation/requirements.md` (full text): the directly-depended-upon upstream feature's approved requirements (instruction-list representation, 15 operators, no `d_dt`, free coefficients).
- `.reviewcompass/specs/term-variation-operators/requirements.md` (target, full text): the document under review.

## Judgment items

Only one independent judgment item is present; no split is needed.

- **item_id**: `term-variation-operators-requirements-vertical-intent-transfer`
- **question**: Does `requirements.md` for `term-variation-operators` faithfully inherit (a) the project-level purpose/boundaries/forbidden-actions from intent, and (b) the term/instruction-list structure and constraints already approved in `term-representation/requirements.md`, without omission, weakening, unsupported addition, or drift? Additionally, does it correctly reflect the user's explicit direction that indexing/counting language must not presuppose a 0-based (Python) or 1-based (Julia) scheme?
- **target**: `.reviewcompass/specs/term-variation-operators/requirements.md`
- **source materials**: `intent/INTENT.md`, `intent/NON_GOALS.md`, `intent/DESIGN_PRINCIPLES.md`, `stages/feature-partitioning/2026-07-02-proposal.md` (excerpt), `.reviewcompass/specs/term-representation/requirements.md`
- **out of scope**: `design.md`/`tasks.md` for `term-variation-operators` (do not exist yet), the correctness of `equation-fitting`/`evolution-loop-and-selection`/`term-importance` beyond the stated boundary, the concrete crossover/mutation probability values and indexing scheme (both explicitly deferred to design by the target document itself).

## Split assessment

No split needed. All four functional requirements (seeding, crossover, mutation, generation loop) trace to a single upstream concern: correctly operationalizing the approved instruction-list representation into a generation algorithm, sourced from Koga's 2018 thesis (section 3.1.1), without presupposing a specific programming language's indexing convention.

## Model-readable source material to include

Full text of the three intent documents, the feature-partitioning excerpt, and the full text of `term-representation/requirements.md` (the upstream feature this one depends on) must be inlined, not path-only, since the reviewer must verify structural consistency with the already-approved representation (operator set, sequential-execution model, free-coefficient handling).

## Sensitive information check

None of the source materials or the target document contain API keys, tokens, passwords, personal identifiers, or third-party confidential material.

## Unresolved, unread, or speculative items

- `design.md` and `tasks.md` do not exist yet; expected at this stage.
- The concrete crossover/mutation probabilities and the 0- vs 1-based indexing scheme are intentionally left to design, per explicit user direction in this session; the target document states this deferral explicitly rather than speculating about values.
