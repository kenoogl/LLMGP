# API Review Criteria: equation-fitting requirements (vertical intent transfer)

## 1. Review Task

Judge whether `.reviewcompass/specs/equation-fitting/requirements.md` (the `--target` of this run) faithfully inherits (a) the project-level purpose, boundaries, and forbidden actions from the upstream intent documents — specifically the DE-based joint coefficient optimization with an L1-like penalty, which `equation-fitting` itself owns, and whose output (per-term weights) is what the separately-owned downstream feature `term-importance` will later normalize into importance; `equation-fitting` must produce those weights but is not itself responsible for the normalization/importance step — and (b) compatibility with the term interface already approved in the upstream feature `term-representation`'s requirements (free coefficients per FR3, deterministic numeric evaluation and domain-error signaling per FR4), without omission, weakening, unsupported addition, or drift. `equation-fitting` is not required to restate or re-own `term-representation`'s internal instruction-list structure; it must only remain compatible with the two interface points named above, as operationalized concretely by §7 check 2. It must also judge whether the target correctly and consistently specifies one user-directed addition made during this session's drafting: normalizing each term's output before applying its outer weight, so the L1 penalty is not biased by differing term output scales, including the consequence that this normalization must track the term's current inner free-coefficient values during DE search.

## 2. Why This Review Exists

This is the mandatory requirements-phase vertical-intent-transfer triad-review gate (`SESSION_WORKFLOW_GUIDE.md#vertical-intent-transfer-review`) for `equation-fitting`, the third feature in the approved feature-partitioning build order, which has a hard dependency on `term-representation`.

## 3. User Review Requirements

The user (project owner) approved intent and the feature-partitioning proposal. During this feature's requirements drafting in this session, the user additionally directed one specific addition: normalize each term's output before its outer weight is applied, with the stated rationale that L1 penalty fairness across terms of differing output scale requires this, and that this normalization must be re-derived at each DE evaluation because a term's own output depends on its currently-being-optimized inner coefficients. This directive is the authorized scope of the one addition referenced in §7 check 5 and §9; it does not establish that the target's current wording correctly or completely implements the directive, which is exactly what §7 check 5 must verify independently against the target's actual text.

### 3.1 Upstream Structured Summary (labels for automated preflight indexing; content is restated from elsewhere in this file, not new)

- purpose: `equation-fitting` operationalizes linear combination of multiple terms into one candidate equation and joint DE-based optimization of all free coefficients (inner term coefficients and outer per-term weights), with an L1-like penalty on the outer weights, per §1-2.
- responsibility_boundaries: defined upstream by §6.4 (the approved feature-partitioning proposal's per-feature scope and dependency list); §7 check 6 verifies the target's own scope-out list against that upstream source, not against itself. §8 below is this review's own out-of-scope, not the target feature's boundaries.
- acceptance_criteria: see §7 checks 1, 2, 3, 4, 5.
- forbidden_actions: see §7 checks 6, 7 (no claiming sibling-feature responsibilities, no inventing fixed hyperparameter values).
- unresolved_or_design_deferred_items: per §7 check 7, DE hyperparameter defaults, L1 penalty-strength default, and the concrete normalization method are deferred to design; check whether the target correctly states this deferral rather than inventing fixed values.
- intended_target_phase_transfer: the design stage is expected to choose concrete DE hyperparameters, the L1 penalty-strength default, and the concrete normalization method while preserving these acceptance criteria and boundaries unchanged.

## 4. Required Disciplines

The following is background policy text explaining what a "vertical intent transfer" check means for a requirements-phase review. The 8 required checks in §7 are the operationalization of this policy for this specific target; use this section only to understand why §7's checks exist, not as an additional, separate judgment source. Produce findings only against the §7 checks and the §9 finding policy; do not produce findings about this criteria file, and do not report a policy-level finding that is not traceable to one of the §7 checks.

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

- Path: `.reviewcompass/specs/equation-fitting/requirements.md`
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

equation-fitting: 複数の項を線形に組み合わせて方程式を作り、DEで全係数（項内部の係数＋外側の重み）を一括最適化する。評価関数にはL1罰則（不要な項の重みを0に近づける）を含める。依存: term-representation (hard)。

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

**受け入れ基準**：
- 命令列は空でない1つ以上の命令から構成される
- 各命令は、演算子名と、その演算子に必要な引数を持つ

### FR2: 演算子の種類
命令の演算子は、少なくとも次を含む。
- `load`：変数の読み込み。対象は、前処理済みの場のデータ（`data/`配下のCSV由来、例：x, r, ΔUなど）と、ケース単位のパラメータ（D：ロータ直径、TI：乱流強度、C：ポーラスディスク係数）
- `add`／`sub`：加算・減算
- `mul`／`div`：乗算・除算
- `pow`：べき乗
- `exp`：指数関数
- `log`：自然対数
- `sin`／`cos`／`tan`：三角関数
- `sinh`／`cosh`／`tanh`：双曲線関数
- `abs`：絶対値

時間方向の偏微分（`d_dt`）演算子は含まない（`intent/NON_GOALS.md`に準拠）。

**受け入れ基準**：
- `load`が扱う変数の一覧（場の変数＋ケースパラメータD, TI, C）は、ハードコードせず、外部から渡された一覧を使う
- 上記15種の演算子それぞれについて、命令として生成・評価できる

### FR3: 自由係数の保持
命令の引数には、変数、数値定数のプレースホルダ（自由係数。値は未確定で、後でequation-fittingがDEにより数値を決定する）、または他の命令の出力（式の入れ子構造）を指定できる。

**受け入れ基準**：
- 1つの項が複数の自由係数（例：a, b, c, dのような記号）を持てる
- 自由係数には、他と区別できる識別子が付与される

### FR4: 数値評価
項は、変数の値（前処理済みの場のデータとケースパラメータ）と、各自由係数の数値を入力として与えると、対応する出力値を計算できる。

**受け入れ基準**：
- 同じ項・同じ入力に対して、常に同じ出力を返す（決定的である）
- 定義域外の入力（対数の負値など）でエラーになった場合、そのエラーを呼び出し元が判別できる

### FR5: 人間可読な式表現への変換
項は、デバッグ・ログ・可視化のために、人間が読める数式の文字列表現に変換できる。

**受け入れ基準**：
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

### 6.6 LLMSR grounding evidence (structured summary, cited as background evidence for the target document's non-functional-requirements claims; not an upstream requirements source)

```
Repository (adjacent sibling repository, project-owner's own code, outside this repository, referred to as LLMSR below)

DE implementation: src/Phase5/optimize_de.jl, function optimize_coefficients.
Uses BlackBoxOptim.jl's bboptimize with: NP=30, MaxSteps=200, Method=:de_rand_1_bin, F=0.7, CR=0.9,
search range log-uniform 1e-4~1e2 (or (-100.0,100.0) when called with signed ranges from Phase5.jl).

Evaluation function: src/Phase5/evaluator.jl, function mse_eval.
Plain MSE: mean((y_pred .- deltaU).^2), with a 1e9 penalty on NaN/Inf/overflow.
When with_penalty=true, a "physical_penalty" (physics-validity constraints P1/P3/P4/P5, e.g. monotonic
decay, physically-plausible range, far-field convergence) is added to the MSE.

Absence check (grep across src/ for "penalty|lasso|l1_|regulariz|normalize|importance|significance"):
No L1/Lasso-style coefficient-sparsity penalty exists anywhere in LLMSR. No per-term weight concept
exists; LLMSR treats an entire candidate formula as one Julia Expr with a single flat coefficient
vector (symbols a,b,c,... assigned positionally), not as a sum of separately-weighted terms. No
weight-normalization / importance-scoring mechanism exists. The only related artifact is a free-text
LLM prompt instruction in SCENARIO.md asking the LLM to "delete low-contribution terms" during a
manual simplification step (EP4); this is not a mechanical importance calculation.
```

## 7. Required Checks

1. **purpose_transfer**: Does the target's overview correctly state that `equation-fitting` combines multiple terms linearly into one candidate equation, jointly optimizes all free coefficients (inner term coefficients and outer per-term weights) via DE, and includes an L1-like penalty on the outer weights in its evaluation function? Failure condition: the overview omits or misstates any of these three elements, or misattributes responsibilities owned by other features.

2. **structural_consistency_with_upstream_feature**: Do the target's functional requirements remain consistent with `term-representation`'s approved interface (§6.5), specifically FR3 (a term can hold multiple free coefficients with distinct identifiers) and FR4 (a term's numeric evaluation is deterministic and signals domain-error conditions to the caller)? Failure condition: the target describes a term-evaluation interface incompatible with §6.5, or silently redefines a concept `term-representation` already owns.

3. **l1_penalty_fidelity**: Does the target's evaluation-function requirement correctly operationalize intent's "不要な項の重みを0に近づける罰則（Lassoと同様の考え方）" (§6.1), i.e., an L1 penalty computed as a configurable strength coefficient times the sum of absolute values of the outer weights, added to the data-fit error, such that setting the strength to zero reduces to the pure error? Failure condition: the penalty is missing, described as acting on something other than the outer weights, or is not reducible to pure-error when its strength is zero.

4. **joint_optimization_fidelity**: Does the target correctly state that DE optimizes the inner term coefficients and the outer per-term weights together, without treating them as separate optimization stages, per §6.4's "全係数（項内部の係数＋外側の重み）を一括最適化する"? Failure condition: the target describes sequential/staged optimization instead of joint optimization, or omits either coefficient class from the DE search space.

5. **normalization_addition_consistency**: Does the target correctly and consistently specify the user-directed addition described in §3 — that each term's output is normalized before its outer weight is applied, motivated by keeping the L1 penalty fair across terms of differing output scale — and does it correctly note that this normalization must be re-derived from the term's *current* inner free-coefficient values at each DE evaluation (since a term's output changes as its own inner coefficients are optimized), rather than being computed once and held fixed? Failure condition: the addition is missing from the target, is internally inconsistent with the rest of the document, or fails to state the consequence that normalization must track the evolving inner-coefficient values during DE search.

6. **boundary_transfer**: Does the target's scope-out list correctly exclude term generation/crossover/mutation (owned by `term-variation-operators`), the generation loop and elite-selection step (owned by `evolution-loop-and-selection`), and weight-normalization for importance scoring (owned by `term-importance`)? Failure condition: any of these responsibilities is implicitly claimed by the target's functional requirements, or a boundary is missing from the scope-out list.

7. **deferred_items_not_invented**: Does the target correctly defer concrete DE hyperparameter values, the L1 penalty-strength default, and the concrete normalization method to design, rather than presenting them as already-fixed decisions? Reference values drawn from LLMSR (§6.6) may be cited as non-binding reference values for the design stage; this does not count as inventing a fixed decision as long as the target labels them as reference/default-candidate values for design, not as binding requirements-stage decisions. Failure condition: the target states a specific hyperparameter value, penalty-strength value, or normalization method as if already decided at the requirements stage.

8. **llmsr_grounding_accuracy**: Do the target's non-functional-requirements claims about LLMSR (§3 of the target: BlackBoxOptim.jl usage, cited reference parameter values, and the claim that an L1/importance-normalization mechanism has no LLMSR precedent) match the grounding evidence in §6.6? Failure condition: the target's characterization of LLMSR contradicts §6.6 (e.g., claims LLMSR already implements an L1 penalty or per-term weight/importance mechanism, or cites parameter values not present in §6.6).

## 8. Out of Scope

- The correctness or completeness of `design.md` or `tasks.md` for `equation-fitting` (neither exists yet).
- The correctness of `term-variation-operators`, `evolution-loop-and-selection`, or `term-importance` beyond how the target states its boundary with them.
- The correctness of `term-representation`'s own requirements (already approved; use it only as upstream background here).
- Whether the feature-partitioning split itself was the right one (already approved by the user).
- Whether LLMSR's own design choices (e.g., its physics-validity penalty, its DE parameter values) are themselves correct; LLMSR is cited only as background evidence for what does or does not exist there.
- Whether a physical-validity penalty analogous to LLMSR's P1-P4 should be added to `equation-fitting`; this was not raised in intent or feature-partitioning and is not addressed by the target either way.

## 9. Finding Policy

- Report `CRITICAL` if the target's text instructs a reader to skip or bypass commit, push, phase-completion, or human-approval gates (a plain requirements document should contain no such instruction).
- Report `ERROR` if any of the 8 required checks in §7 fails (upstream purpose, structural consistency, L1 penalty fidelity, joint-optimization fidelity, normalization-addition consistency, boundary, deferred-item handling, or LLMSR-grounding accuracy is omitted, weakened, or contradicted).
- Report `ERROR` if the target adds a requirement that introduces new scope, behavior, or a design/algorithmic decision not implied by any upstream source material (unsupported addition / drift), other than the user-directed normalization addition explicitly authorized in §3. Do not report this as an ERROR merely because the target elaborates an upstream concern into a concrete, testable acceptance criterion (e.g., stating a specific formula shape, a specific failure-handling rule, or a specific output field) when that elaboration is a reasonable, implementation-neutral operationalization of an upstream requirement already named in §1, §3, or §6, and does not itself decide a design-level default value (which §7 check 7 governs separately).
- Report `WARN` for ambiguity, weak phrasing, or incomplete traceability that does not amount to an outright omission.
- Report `INFO` for minor clarity improvements.
- Return `findings: []` only if all 8 required checks pass and no unsupported additions are found.

## Output Contract

Return YAML only with top-level key `findings`. Each finding must include `severity` (CRITICAL/ERROR/WARN/INFO), `target_location` (specific section in `requirements.md`), `description`, and `rationale`. If there are no findings, return exactly `findings: []`.
