# Main Preanalysis: term-representation requirements review

## Source materials read and purpose

- `intent/INTENT.md` (full text): upstream purpose, the 6 planned enhancements, the GP-process decisions (no `d_dt`, expressive terms, DE+L1-penalty term selection).
- `intent/NON_GOALS.md` (full text): explicit exclusions (no LLMSR rewrite-in-place, no CFD simulation, no LLM training, no time-derivative equation discovery).
- `intent/DESIGN_PRINCIPLES.md` (full text): staged development (LLM-less GP core first), no-LLM-GP is LLMGP's own design (not a literal reproduction of Koga & Ono 2019).
- `stages/feature-partitioning/2026-07-02-proposal.md` (excerpt for `term-representation`): approved scope and dependency (`term-representation` has no dependency; it is the foundation for `term-variation-operators` and `equation-fitting`).
- `.reviewcompass/specs/term-representation/requirements.md` (target, full text): the document under review.

## Judgment items

Only one independent judgment item is present; no split is needed.

- **item_id**: `term-representation-requirements-vertical-intent-transfer`
- **question**: Does `requirements.md` for `term-representation` faithfully inherit the purpose, responsibility boundaries, acceptance criteria, and forbidden actions established in the upstream intent documents, without omission, weakening, unsupported addition, or drift?
- **target**: `.reviewcompass/specs/term-representation/requirements.md`
- **source materials**: `intent/INTENT.md`, `intent/NON_GOALS.md`, `intent/DESIGN_PRINCIPLES.md`, `stages/feature-partitioning/2026-07-02-proposal.md` (term-representation section)
- **out of scope**: `design.md` / `tasks.md` (do not exist yet; downstream correctness is not judged here), the correctness of other features (`term-variation-operators`, `equation-fitting`, `evolution-loop-and-selection`, `term-importance`) beyond how `term-representation` states its boundary with them.

## Split assessment

No split needed. All acceptance criteria in the target document (FR1-FR5) trace back to a single upstream concern: does the term/gene data structure preserve LLMSR-level expressiveness while excluding what intent explicitly put out of scope.

## Model-readable source material to include

Full text of the three intent documents and the relevant feature-partitioning excerpt should be inlined in the criteria file (not path-only references), since the reviewer must independently verify the mapping from these upstream statements into the target's functional requirements and scope-out list.

## Sensitive information check

None of the source materials or the target document contain API keys, tokens, passwords, personal identifiers, or third-party confidential material. All materials are internal project specification documents that the user has already reviewed and approved (intent approval, feature-partitioning approval) in this session, and the user has authorized this API review-run.

## Unresolved, unread, or speculative items

- `design.md` and `tasks.md` for `term-representation` do not exist yet; this is expected at the requirements-drafting stage and is not a gap.
- The concrete crossover/mutation algorithm (`term-variation-operators`) and the exact DE fitness function with L1-style penalty (`equation-fitting`) are intentionally deferred to their own features per the approved feature-partitioning; `requirements.md` correctly lists these as out of scope rather than speculating about them.
