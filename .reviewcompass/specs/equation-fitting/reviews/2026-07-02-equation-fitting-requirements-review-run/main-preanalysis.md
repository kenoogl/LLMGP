# Main Preanalysis: equation-fitting requirements review

## Source materials read and purpose

- `intent/INTENT.md`, `intent/NON_GOALS.md`, `intent/DESIGN_PRINCIPLES.md` (full text): project-level purpose, exclusions, staged-development principle, and the specific intent sentence describing DE-based coefficient optimization with an L1-like penalty and normalized-weight importance.
- `stages/feature-partitioning/2026-07-02-proposal.md` (excerpt): approved scope for `equation-fitting` ("複数の項を線形に組み合わせて方程式を作り、DEで全係数（項内部の係数＋外側の重み）を一括最適化する。評価関数にはL1罰則を含める") and its dependency (hard depends on `term-representation` only).
- `.reviewcompass/specs/term-representation/requirements.md` (full text): the directly-depended-upon upstream feature's approved requirements — instruction-list structure, free coefficients (FR3), deterministic numeric evaluation and domain-error signaling (FR4).
- `.reviewcompass/specs/equation-fitting/requirements.md` (target, full text): the document under review.
- LLMSR reference code (`/Users/Daily/Development/WindTurbineWake/LLMSR`, outside this repository, cited as background evidence only, not as an upstream requirements source): `src/Phase5/optimize_de.jl` (DE via BlackBoxOptim.jl, `NP=30, MaxSteps=200, Method=:de_rand_1_bin, F=0.7, CR=0.9`, search range log-uniform 1e-4~1e2) and `src/Phase5/evaluator.jl` (`mse_eval` = plain MSE, `physical_penalty` = additive physics-validity penalty, no L1/Lasso-style coefficient-sparsity mechanism, no per-term weight or importance-normalization mechanism). This grounds the target document's non-functional-requirements claims that (a) LLMSR's DE parameters are usable as reference defaults and (b) the L1 penalty and importance-normalization mechanisms have no LLMSR precedent and are new design work.

## Judgment items

Only one independent judgment item is present; no split is needed.

- **item_id**: `equation-fitting-requirements-vertical-intent-transfer`
- **question**: Does `requirements.md` for `equation-fitting` faithfully inherit (a) the project-level purpose/boundaries/forbidden-actions from intent, specifically the DE-based joint coefficient optimization with an L1-like penalty and normalized-weight importance, and (b) the term/instruction-list structure and interface already approved in `term-representation/requirements.md` (free coefficients, deterministic numeric evaluation, domain-error signaling), without omission, weakening, unsupported addition, or drift? Additionally, does it correctly and consistently specify the user-directed addition made during this session's drafting — normalizing each term's output before applying its outer weight, so that the L1 penalty is not biased by differing term output scales — including the consequence that this normalization must track the term's current inner free-coefficient values during DE search (since a term's output changes as its own inner coefficients are optimized)?
- **target**: `.reviewcompass/specs/equation-fitting/requirements.md`
- **source materials**: `intent/INTENT.md`, `intent/NON_GOALS.md`, `intent/DESIGN_PRINCIPLES.md`, `stages/feature-partitioning/2026-07-02-proposal.md` (excerpt), `.reviewcompass/specs/term-representation/requirements.md`, and the structured LLMSR findings summarized above.
- **out of scope**: `design.md`/`tasks.md` for `equation-fitting` (do not exist yet); the correctness of `term-variation-operators`, `evolution-loop-and-selection`, or `term-importance` beyond the stated boundary; the concrete DE hyperparameter defaults, the concrete L1 penalty-strength default, and the concrete normalization method (all explicitly deferred to design by the target document itself); whether LLMSR's own implementation choices are correct (LLMSR is cited only as background evidence, not as a standard to certify).

## Split assessment

No split needed. All five functional requirements (equation assembly with output normalization, joint DE optimization, error+L1 objective function, evaluation-failure handling, result retrieval) trace to a single upstream concern: correctly operationalizing `equation-fitting`'s approved responsibility (linear combination of terms + joint coefficient optimization + L1 penalty) into testable requirements that are consistent with `term-representation`'s already-approved term interface, plus verifying the one user-directed addition (pre-weighting normalization) is specified clearly and consistently within that same document.

## Model-readable source material to include

Full text of the three intent documents, the feature-partitioning excerpt for all five features (needed to verify the scope-out boundaries against sibling features), and the full text of `term-representation/requirements.md` (the upstream feature this one depends on) must be inlined, not path-only, since the reviewer must verify structural consistency with the already-approved term interface (FR3 free coefficients, FR4 deterministic evaluation and domain-error signaling). The LLMSR grounding material is included as a short structured summary (specific function names, file paths, and the absence of an L1/importance mechanism), not the full source files, since only the target document's factual characterization of LLMSR needs verification, not LLMSR's own design.

## Sensitive information check

None of the source materials or the target document contain API keys, tokens, passwords, personal identifiers, or third-party confidential material. The LLMSR excerpts are this project owner's own adjacent-repository code, not third-party confidential material.

## Unresolved, unread, or speculative items

- `design.md` and `tasks.md` do not exist yet; expected at this stage.
- The concrete DE hyperparameters, L1 penalty-strength default, and normalization method are intentionally left to design; the target document states this deferral explicitly (§3 non-functional requirements, FR1 acceptance criteria) rather than speculating about values.
- Whether a physical-validity penalty (analogous to LLMSR's P1-P4) should be added to `equation-fitting` was not raised in intent or feature-partitioning and is treated as out of scope for this review; it is not mentioned in the target document either way.
