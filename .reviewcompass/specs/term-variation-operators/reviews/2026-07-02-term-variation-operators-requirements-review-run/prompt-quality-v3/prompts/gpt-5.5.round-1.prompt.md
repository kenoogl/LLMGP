prompt_id: openai_review
provider: openai-api
model_id: gpt-5.5

# Task
Review the target document for the requested phase and criteria.

# Phase
prompt_quality

# Criteria
---
criteria_id: api-review-prompt-quality-review
phase: prompt-quality
review_target: <api-review-criteria-draft.md>
---

# API Review Prompt Quality Review Criteria

Review the target prompt draft itself. Do not review the eventual feature artifact yet.

The question is:

Is the draft prompt suitable to use as the `--criteria-file` for the intended API-mediated review, given that the actual `--target` will be the target artifact named in the draft?

If user-provided review requirements exist, also judge whether the draft prompt preserves and operationalizes those requirements without unauthorized narrowing, broadening, replacement, or added authority.

## Required Quality Checks

Check both general LLM-as-Judge prompt quality and any workflow / phase-specific review requirements.

### General API Review Prompt Quality

The prompt draft must:

1. Show that main identified the materials and judgment points before sending the prompt.
2. Include enough background and related context for the model to avoid guessing.
3. Ask a non-leading analysis question.
4. Define a scope that is neither too broad nor too narrow.
5. Provide or rely on a fixed, parser-friendly output contract when combined with the API review runner template.
6. Avoid credentials, personal data, or unrelated confidential material.
7. Keep `--criteria-file` and `--target` roles distinct.
8. Ensure the target manifest will contain the actual artifact under review.

### User Review Requirements Fit

When the review was requested by a user or by a workflow-specific gate, the prompt draft must:

1. Preserve the requested review purpose, review object, focus, scope boundaries, source materials, output requirements, and prohibited actions.
2. Map those requirements into the review task, required checks, out of scope, and finding policy.
3. Avoid narrowing the requested review so that important user-stated concerns disappear.
4. Avoid broadening the requested review into unrelated judgments or downstream phase decisions.
5. Avoid replacing the requested review question with a more convenient or generic question.
6. Mark assumptions explicitly when the user requirement is ambiguous.
7. Keep workflow-required checks distinguishable from user-requested checks when both apply.
8. Prevent commit, push, phase completion, human approval delegation, or unapproved specification changes unless those actions are explicitly in scope and allowed by the governing workflow.

### Judgment Granularity

The prompt draft must:

1. Contain one primary judgment question.
2. Avoid bundling multiple independent cluster, finding, acceptance, or design-policy judgments into one prompt.
3. Split multiple independent judgments into separate prompt files unless they are inseparable parts of the same decision.
4. Include only the source findings, upstream materials, target summaries, and output fields needed for the specific judgment item.
5. Keep common background short enough that it does not obscure the item-specific evidence.
6. Require separate prompt-quality review evidence for each item-specific prompt when the review is split.

### Source-Material Quality

The prompt draft must:

1. Clearly distinguish review target, source materials, and out-of-scope material.
2. Avoid path-only source material listing.
3. Include required source material as excerpt or structured summary in a model-readable form.
4. Mark front matter paths as provenance when they are not model-readable content.
5. Include unresolved or deferred items when they affect the target review.

### External API Material Policy

The prompt draft may include ReviewCompass repository specifications, designs, tasks, review findings, structured summaries, and evidence paths when they are necessary for API review or proxy_model judgment and the user has approved that API/proxy execution.

The prompt draft must:

1. Exclude API keys, tokens, passwords, nonces, and other secrets.
2. Exclude personal identifiers such as email addresses and phone numbers unless explicitly required and approved.
3. Exclude third-party confidential material that is not allowed to be sent externally.
4. Avoid unnecessary full logs or unrelated files.
5. Use the minimum review-relevant excerpts or structured summaries needed for the judgment item.
6. Not treat ordinary repository-internal specs, designs, review findings, or evidence paths as automatically prohibited solely because they are unpublished.

### Vertical Intent Transfer Requirements

When the review is a phase review under `SESSION_WORKFLOW_GUIDE.md#vertical-intent-transfer-review`, the prompt draft must:

1. Limit target judgment to the current phase artifact.
2. Use upstream artifacts only for background and intent-transfer checking.
3. Include upstream purpose, responsibility boundaries, acceptance criteria, forbidden actions, unresolved items, and intended target-phase transfer.
4. Prevent the model from judging downstream tasks or implementation correctness unless downstream invention would be forced by a target omission.

### Output And Triage Fit

The prompt draft must:

1. Use severity labels compatible with the API parser: `CRITICAL`, `ERROR`, `WARN`, `INFO`.
2. Give enough target-location guidance for findings to be triaged.
3. Define when `findings: []` is acceptable.
4. Avoid wording that suppresses findings about critical boundaries while merely trying to prohibit the model from authorizing operations.

## Finding Policy

- Report `CRITICAL` if the draft prompt would authorize or imply authorization of irreversible operations, gate completion, or human-only approvals.
- Report `CRITICAL` if the draft prompt would violate a user-stated prohibited action or authority limit.
- Report `ERROR` if the draft prompt omits, weakens, broadens, or replaces a user-stated review requirement in a way that changes what will be reviewed.
- Report `ERROR` if the draft prompt combines multiple independent judgments so that the model is likely to miss item-specific evidence, conflate labels, or produce unusable decisions.
- Report `ERROR` if the draft prompt would likely cause models to review the wrong target, rely on summaries instead of the real target, miss required upstream-transfer obligations, leak secrets or prohibited third-party confidential material, or produce unusable output.
- Report `WARN` if the draft prompt is usable but has avoidable ambiguity, framing bias, incomplete source-material presentation, weak target/source/out-of-scope separation, incomplete mapping from user requirements to checks, or mildly oversized common background.
- Report `INFO` for minor wording or ergonomics improvements.
- Return `findings: []` only if the draft prompt is safe to use for the intended API review.

## Output Contract

Return YAML only with top-level key `findings`.

Each finding must include:

- `severity`: CRITICAL, ERROR, WARN, or INFO
- `target_location`: specific section in the prompt draft
- `description`: concise finding summary
- `rationale`: why this matters for API review quality

If there are no findings, return exactly:

findings: []


# Output contract
Return YAML only.
The response must include the top-level key findings.
Additional top-level keys are allowed only when the criteria explicitly defines them.
Do not add wrapper keys such as review, result, metadata, or summary.
Do not wrap the YAML in Markdown code fences.
Do not write prose before or after the YAML.

Each finding must include these keys:
- severity
- target_location
- description
- rationale

Use only these severity values:
- CRITICAL
- ERROR
- WARN
- INFO

If there are no findings and the criteria does not define additional top-level keys, return exactly:

findings: []

Valid shape example:

findings:
  - severity: WARN
    target_location: "path or section"
    description: "Plain finding summary"
    rationale: "Why this matters"

# Prior findings
なし

# Target path
.reviewcompass/specs/term-variation-operators/reviews/2026-07-02-term-variation-operators-requirements-review-run/review-target.md

# Target document
# API Review Criteria: term-variation-operators requirements (vertical intent transfer)

## 1. Review Task

Judge whether `.reviewcompass/specs/term-variation-operators/requirements.md` (the `--target` of this run) faithfully inherits (a) the project-level purpose, boundaries, and forbidden actions from the upstream intent documents, and (b) the instruction-list structure and constraints already approved in the upstream feature `term-representation`'s requirements, without omission, weakening, unsupported addition, or drift.

The project-level check in (a) is operationalized concretely by §7 check 1 (purpose) and check 2 (structural consistency with `term-representation`, which is where the `d_dt`/steady-state exclusion is already established; term-variation-operators inherits this by using `term-representation`'s instruction set and need not restate it, so check 2 should fail only if the target contradicts this inherited exclusion, e.g. by describing a `d_dt`-like operator). Other `NON_GOALS.md` items (no CFD simulation, no LLM training) are not directly applicable to term-variation-operators; check only that the target does not contradict them, and do not require the target to restate them.

## 2. Why This Review Exists

This is the mandatory requirements-phase vertical-intent-transfer triad-review gate (`SESSION_WORKFLOW_GUIDE.md#vertical-intent-transfer-review`) for `term-variation-operators`, the second feature in the approved feature-partitioning build order, which has a hard dependency on `term-representation`.

## 3. User Review Requirements

The user (project owner) approved intent and the feature-partitioning proposal, and specifically directed the following during this feature's requirements drafting: (a) the seeding/crossover/mutation algorithms should follow Koga's 2018 master's thesis section 3.1.1 (instructions can only reference earlier-generated instructions; crossover swaps a leading prefix of instructions between two parents; mutation replaces one instruction); (b) because the thesis's reference implementation is Python (0-based indexing) while this project targets Julia (1-based indexing by convention), the requirements text must describe ordering/counting constraints without presupposing either indexing scheme; (c) the crossover/mutation probability values and the concrete indexing scheme are explicitly deferred to design, not fixed here.

### 3.1 Upstream Structured Summary (labels for automated preflight indexing; content is restated from elsewhere in this file, not new)

- purpose: `term-variation-operators` operationalizes seeding/crossover/mutation for the GP term population, building on `term-representation`'s instruction-list structure, per §1-2.
- responsibility_boundaries: see §7 checks 2 and 5 (the target document's own scope-out list is the source of truth; §8 below is this review's own out-of-scope, not the target feature's boundaries).
- acceptance_criteria: see §7 checks 1, 3, 4.
- forbidden_actions: see §7 checks 4, 6 (no indexing-scheme presupposition, no probability-value invention).
- unresolved_or_design_deferred_items: per the user direction in §3(c) above, crossover/mutation probability values and the concrete 0- vs 1-based indexing scheme are deferred to design; check whether the target correctly states this deferral (see §7 check 6) rather than inventing values.
- intended_target_phase_transfer: the design stage is expected to choose the concrete Julia indexing scheme and default probability values while preserving these acceptance criteria and boundaries unchanged.

## 4. Required Disciplines

The following is background policy text explaining what a "vertical intent transfer" check means for a requirements-phase review. It is provided so you can judge the target document directly against this policy (in addition to the required checks in §7), not so you evaluate this criteria file itself. Do not produce findings about this criteria file; all findings must be about the target document named in §5.

### 4.1 SESSION_WORKFLOW_GUIDE.md#vertical-intent-transfer-review (full text of the cited section, background only)

```
### 2.2.1 上流意図伝達の必須検査

各 phase の triad-review／review-wave／alignment では、対象 phase の成果物だけでなく、上流成果物または上流判断材料からの意図伝達を必須検査項目とする。review prompt は、少なくとも「上流の目的・責務境界・受入条件・禁止事項が、対象成果物へ欠落・弱体化・逸脱・未根拠追加なく引き継がれているか」を問わなければならない。

- requirements review：上流判断材料 → requirements.md を確認し、reopen 分類根拠、利用者判断、計画メモ、設計メモなどの目的・責務境界・受入条件・禁止事項が要件へ欠落なく落ちているかを検査する。design.md / tasks.md は参照資料であり、審査対象ではない
- design review：requirements.md → design.md を確認し、要件の目的・境界・受入条件が設計へ欠落なく落ちているかを検査する
- tasks review：requirements.md → design.md → tasks.md を確認し、要件と設計の意図が implementation-ready なタスク粒度へ落ちているかを検査する
- implementation review：requirements.md → design.md → tasks.md → implementation を確認し、実装がタスクだけでなく上流意図から逸脱していないかを検査する
```

(English gloss: for the requirements phase specifically, the review must check that the target's purpose/boundaries/acceptance-criteria/forbidden-actions are inherited from upstream decision materials without omission; `design.md`/`tasks.md` are reference-only, not judged.)

## 5. Review Target

- Path: `.reviewcompass/specs/term-variation-operators/requirements.md`
- This file is supplied to you directly via the `--target` argument of the review runner; judge its actual full text, not a summary of it.

## 6. Source Materials

Use these source materials as background and for intent-transfer verification only. Do not judge their correctness; judge only whether the target correctly reflects them.

### 6.1 intent/INTENT.md (upstream, full text)

```
# LLMGP Intent

## 背景

隣接プロジェクト LLMSR は、風車後流のCFD（数値流体力学）シミュレーション結果から、速度欠損 ΔU(x, r) を表す代数式モデルを自動探索するJulia製の研究基盤である。LLMが構造式（数式）の生成・交叉・変異を行う疑似的なGP（遺伝的プログラミング）として構成されており、戦略はPhase5・Phase6の2つのワークフロー文書（WORKFLOW_PHASE5.md、WORKFLOW_PHASE6.md）に記述されている。trial_1〜trial_10まで試行済み。

## 目的

LLMGPは、LLMSRの作り直し（別バージョン）である。LLMSRのJuliaコードは直接コピーせず、参照しながら新規に書き直す。GPプロセスと評価軸を精査し、次の機能を中心に改良・追加する。

1. **複数ケース間の汎化性能評価**：異なる条件（乱流強度など）のデータで発見したモデルが、他の条件でも通用するかを評価できるようにする
2. **複数LLMの比較**：異なるLLM（例：Gemini、GPT、Claudeなど）による構造式生成・進化戦略の性能を比較できるようにする
3. **対照実験としてのGPのみ実行**：最初に構築するLLMなしGPフレームワーク（後述）そのものを、LLM併用時との比較対象として使う
4. **収束履歴の可視化**：進化過程（世代ごとのスコア推移など）の可視化を強化する
5. **複数の評価地点**：単一の集約スコアだけでなく、複数の評価地点での精度を評価できるようにする。評価地点は x/D = 2, 5, 10, 15（xは風車から流れ方向＝下流方向への距離、Dはロータ直径）。直接の評価地点ではないが、後流幅（wake width、後流が広がる範囲）と中心速度欠損（wake centerline deficit、後流の中心軸上での速度の落ち込み）も評価の観点に加える
6. **既存モデルとのベンチマーク比較**：発見したモデルを、既存の後流モデル（Jensen、BP＝Bastankhah and Porté-Agel、Larsen、Frandsen）と比較する。いずれも数値計算の繰り返しが不要な閉じた式。参考文献：Neiva et al. (2019) "A Review of Wind Turbine Wake Models"。Ainslie（Eddy-Viscosityモデル）とFugaモデルは、乱流の輸送方程式を数値的に解く必要があり実装コストが大きいため、今回は後回しにする。

GPプロセス自体の精査点：時間方向の偏微分（d_dt演算子）は扱わない（対象は定常な代数式）。項（遺伝子）は、LLMSRと同様に内部に自由な係数（b, c, dなど）を持てる表現力の高い形式を維持する。項の自動選択と重要度の算出は、LLMSRから流用するDE（差分進化法）による係数最適化の評価関数に、不要な項の重みを0に近づける罰則（Lassoと同様の考え方）を加える方式を採用する。最適化後の各項の重みを正規化した値を、項の重要度として扱う。

## 対象データ

`data/`配下の風車後流CFDシミュレーション結果（座標・速度・圧力・乱流量を含むCSV）。

今後、乱流強度 I（turbulence intensity） = 0.01, 0.03, 0.05, 0.1, 0.2, 0.3 と、C（係数）= 10, 13, 16, 19, 22, 25 の組み合わせでケースを整備予定（最大36ケース）。現時点では I=0.1・C=16、I=0.1・C=22 の2ケースのみ配置済み。この複数ケース群が、機能1「複数ケース間の汎化性能評価」の入力データとなる。
```

### 6.2 intent/NON_GOALS.md (upstream, full text)

```
# LLMGP でやらないこと

- LLMSR（隣のプロジェクト）そのものを直接書き換えることはしない。LLMGPは別のプロジェクトとして、独立して作る。
- 風の流れを計算するシミュレーション（CFD）自体は、このプロジェクトでは行わない。すでに計算済みのデータファイル（data/配下のCSV）を使う。
- LLM（AIモデル）そのものを新しく作ったり訓練したりはしない。すでにあるLLMのサービスを呼び出して使う側に徹する。
- 時間方向の偏微分を扱う方程式の発見はしない（対象は定常な代数式）。GPプロセスの詳細はINTENT.mdを参照。
```

### 6.3 intent/DESIGN_PRINCIPLES.md (upstream, full text)

```
# LLMGP 設計の考え方

- LLMSRのコードはそのままコピーせず、内容を参考にしながら新しく書き直す。
- 式の生成・入れ替え・入れ替えのもとになる操作（GPでいう「交叉」「変異」など）を、はっきりした手順として書く。あいまいなままにしない。
- 新しい評価の仕組み（複数の条件で試す、複数のLLMを比べる、複数の場所で精度を見る、など）を後から足しやすい作りにする。
- LLMを使う場合と使わない場合（GPだけの対照実験）を、同じ仕組みの中で切り替えて実行できるようにする。
- 開発は段階的に進める。まずLLMを使わない自動処理（GPの交叉・変異、DEによる係数最適化、項の自動選択）だけでフレームワーク全体を作り、想定する機能（複数ケースでの評価、複数評価地点、収束履歴の可視化など）が一通り動くことを確認する。LLMの導入はそのあとの段階で行う。
- 「LLMなしGP」は、元論文（古賀・小野2019）の忠実な再現ではなく、LLMGP独自の設計（DEの評価関数への罰則導入など）によるLLMなし版である。対照実験でもこのLLMなしGPを使う。
```

### 6.4 stages/feature-partitioning/2026-07-02-proposal.md (upstream, relevant excerpt)

```
term-representation: 項（遺伝子）のデータ構造。命令列（load・mul・div・exp・log・べき乗などの演算子）と、内部に自由な係数（b, c, dなど）を持てる形式を定義する。依存先なし。

term-variation-operators: 項の初期生成（seeding）、交叉（1点交叉）、変異（命令の一部をランダムに変更）。依存: term-representation (hard)。

equation-fitting: 複数の項を線形に組み合わせて方程式を作り、DEで全係数を一括最適化する。評価関数にはL1罰則を含める。依存: term-representation (hard)。

evolution-loop-and-selection: 世代交代のループ本体。子の生成（term-variation-operatorsを使用）→方程式の組み立てと評価（equation-fittingを使用）→エリート戦略による選択、を繰り返す。依存: term-variation-operators (hard)、equation-fitting (hard)。

term-importance: 最適化後の項の重みを正規化し、重要度を算出する。依存: equation-fitting (hard)。
```

### 6.5 .reviewcompass/specs/term-representation/requirements.md (directly-depended-upon upstream feature, full text)

```
# term-representation 要件定義

## 1. 概要

GP（遺伝的プログラミング）の遺伝子にあたる「項」を表すデータ構造を定義する。項は、風車後流モデルの式（例：a * exp(-b*x) * (1 + c*r^2)^(-d)）を構成する部品であり、後続のterm-variation-operators（交叉・変異）とequation-fitting（係数最適化）が操作する対象となる。

## 2. 機能要件

### FR1: 命令列としての表現
項は、逐次実行される命令の並び（リスト）として表現できる。各命令は演算子と引数（オペランド）を持つ。

受け入れ基準：
- 命令列は空でない1つ以上の命令から構成される
- 各命令は、演算子名と、その演算子に必要な引数を持つ

### FR2: 演算子の種類
命令の演算子は、少なくとも次を含む。
- load：変数の読み込み。対象は、前処理済みの場のデータ（data/配下のCSV由来、例：x, r, ΔUなど）と、ケース単位のパラメータ（D：ロータ直径、TI：乱流強度、C：ポーラスディスク係数）
- add／sub：加算・減算
- mul／div：乗算・除算
- pow：べき乗
- exp：指数関数
- log：自然対数
- sin／cos／tan：三角関数
- sinh／cosh／tanh：双曲線関数
- abs：絶対値

時間方向の偏微分（d_dt）演算子は含まない（intent/NON_GOALS.mdに準拠）。

受け入れ基準：
- loadが扱う変数の一覧（場の変数＋ケースパラメータD, TI, C）は、ハードコードせず、外部から渡された一覧を使う
- 上記15種の演算子それぞれについて、命令として生成・評価できる

### FR3: 自由係数の保持
命令の引数には、変数、数値定数のプレースホルダ（自由係数。値は未確定で、後でequation-fittingがDEにより数値を決定する）、または他の命令の出力（式の入れ子構造）を指定できる。

受け入れ基準：
- 1つの項が複数の自由係数（例：a, b, c, dのような記号）を持てる
- 自由係数には、他と区別できる識別子が付与される

### FR4: 数値評価
項は、変数の値（前処理済みの場のデータとケースパラメータ）と、各自由係数の数値を入力として与えると、対応する出力値を計算できる。

受け入れ基準：
- 同じ項・同じ入力に対して、常に同じ出力を返す（決定的である）
- 定義域外の入力（対数の負値など）でエラーになった場合、そのエラーを呼び出し元が判別できる

### FR5: 人間可読な式表現への変換
項は、デバッグ・ログ・可視化のために、人間が読める数式の文字列表現に変換できる。

受け入れ基準：
- 自由係数はその識別子（記号）で表示される（数値ではなく）

## 3. 非機能要件・制約

- 実装言語はJulia（LLMSRのコードを参照する前提のため）
- LLMSRのデータ形式・APIとの互換性は求めない（作り直しのため）

## 4. スコープ外

- 項の交叉・変異・初期生成（seeding）：term-variation-operatorsで扱う
- 自由係数の数値最適化（DE、L1罰則）：equation-fittingで扱う
- 項の重要度算出：term-importanceで扱う
- CSVからのr・ΔUの導出、D・TI・Cの読み込みなどのデータ前処理：別のデータ読み込み機能で扱う（term-representationは前処理済みの変数一覧を受け取るだけ）
```

## 7. Required Checks

1. **purpose_transfer**: Does the target's overview correctly state that `term-variation-operators` handles seeding, crossover, and mutation for the term/gene population defined by `term-representation`? Failure condition: the overview omits or misstates this role, or misattributes responsibilities owned by other features.
2. **structural_consistency_with_upstream_feature**: Do the target's functional requirements remain consistent with `term-representation`'s approved instruction-list structure (§6.5), e.g., not contradicting the operator set, the free-coefficient mechanism, or the sequential-execution model (FR1 of §6.5)? Failure condition: the target describes a term structure incompatible with §6.5, or silently redefines a concept `term-representation` already owns.
3. **algorithm_fidelity**: Do the target's requirements for seeding, crossover, and mutation correctly reflect the three algorithm rules the user approved for this feature, stated in full in §3(a) above (instructions reference only earlier-generated instructions; crossover swaps a leading prefix; mutation replaces one instruction)? These three rules are the complete, user-approved operationalization of the relevant part of Koga's 2018 thesis section 3.1.1 for this project; the thesis section itself is not attached and is not the standard to check against beyond these three rules. Failure condition: any of the three rules is missing, contradicted, or materially altered without justification.
4. **indexing_neutrality**: Does the target avoid presupposing a 0-based or 1-based indexing scheme when describing ordering/counting constraints, per §3(b)? Failure condition: the target uses concrete index arithmetic (e.g., an explicit "[0, i-1]"-style range) that presupposes one indexing convention.
5. **boundary_transfer**: Does the target's scope-out list correctly exclude the term/gene data structure itself (owned by `term-representation`), the elite-selection step (owned by `evolution-loop-and-selection`), coefficient optimization (owned by `equation-fitting`), and importance scoring (owned by `term-importance`)? Failure condition: any of these responsibilities is implicitly claimed by the target's functional requirements, or a boundary is missing from the scope-out list.
6. **deferred_items_not_invented**: Does the target correctly defer the crossover/mutation probability values and the concrete indexing scheme to design, per §3(c), rather than inventing specific values or a specific indexing scheme? Failure condition: the target states a specific probability value or a specific 0-/1-based indexing choice as if already decided.

## 8. Out of Scope

- The correctness or completeness of `design.md` or `tasks.md` for `term-variation-operators` (neither exists yet).
- The correctness of `equation-fitting`, `evolution-loop-and-selection`, or `term-importance` beyond how the target states its boundary with them.
- The correctness of `term-representation`'s own requirements (already approved; use it only as upstream background here).
- Whether the feature-partitioning split itself was the right one (already approved by the user).

## 9. Finding Policy

- Report `CRITICAL` if the target's text instructs a reader to skip or bypass commit, push, phase-completion, or human-approval gates (a plain requirements document should contain no such instruction).
- Report `ERROR` if any of the 6 required checks in §7 fails (upstream purpose, structural consistency, algorithm fidelity, indexing neutrality, boundary, or deferred-item handling is omitted, weakened, or contradicted).
- Report `ERROR` if the target adds a requirement not supported by any upstream source material (unsupported addition / drift).
- Report `WARN` for ambiguity, weak phrasing, or incomplete traceability that does not amount to an outright omission.
- Report `INFO` for minor clarity improvements.
- Return `findings: []` only if all 6 required checks pass and no unsupported additions are found.

## Output Contract

Return YAML only with top-level key `findings`. Each finding must include `severity` (CRITICAL/ERROR/WARN/INFO), `target_location` (specific section in `requirements.md`), `description`, and `rationale`. If there are no findings, return exactly `findings: []`.

