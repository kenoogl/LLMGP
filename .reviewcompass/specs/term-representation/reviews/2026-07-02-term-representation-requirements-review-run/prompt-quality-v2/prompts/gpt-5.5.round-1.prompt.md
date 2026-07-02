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
.reviewcompass/specs/term-representation/reviews/2026-07-02-term-representation-requirements-review-run/review-target.md

# Target document
# API Review Criteria: term-representation requirements (vertical intent transfer)

## 1. Review Task

Judge whether `.reviewcompass/specs/term-representation/requirements.md` (the `--target` of this run) faithfully inherits the purpose, responsibility boundaries, acceptance criteria, and forbidden actions established in the upstream intent documents for the LLMGP project, without omission, weakening, unsupported addition, or drift.

## 2. Why This Review Exists

LLMGP's workflow requires a requirements-phase "vertical intent transfer" check (`SESSION_WORKFLOW_GUIDE.md#vertical-intent-transfer-review`) before `requirements.md` can proceed past drafting. This is the mandatory triad-review gate for the `term-representation` feature, the first and foundation feature of the approved feature-partitioning.

## 3. User Review Requirements

The user (project owner) approved intent (`intent/INTENT.md`, `intent/NON_GOALS.md`, `intent/DESIGN_PRINCIPLES.md`) and the feature-partitioning proposal, and specifically directed several requirements-drafting decisions in conversation: (a) the instruction set must include `load/add/sub/mul/div/pow/exp/log/sin/cos/tan/sinh/cosh/tanh/abs` and exclude `d_dt`; (b) the `load`-able variable set must not be hard-coded but supplied externally, derived from preprocessed field data (`x, r, ΔU`, etc.) plus case parameters D (rotor diameter), TI (turbulence intensity), and C (porous-disk coefficient); (c) deriving `r`/`ΔU` from the raw CSV and loading D/TI/C are explicitly out of scope for this feature (handled by a separate data-loading component). These three decisions must be checked as present and unweakened in the target.

## 4. Required Disciplines

The following are the governing policy texts that §7's required checks were derived from. Use them to independently judge whether §7 correctly and completely operationalizes the general policy, not merely whether the target satisfies §7 as written.

### 4.1 SESSION_WORKFLOW_GUIDE.md#vertical-intent-transfer-review (full text of the cited section)

```
### 2.2.1 上流意図伝達の必須検査

各 phase の triad-review／review-wave／alignment では、対象 phase の成果物だけでなく、上流成果物または上流判断材料からの意図伝達を必須検査項目とする。review prompt は、少なくとも「上流の目的・責務境界・受入条件・禁止事項が、対象成果物へ欠落・弱体化・逸脱・未根拠追加なく引き継がれているか」を問わなければならない。

- requirements review：上流判断材料 → requirements.md を確認し、reopen 分類根拠、利用者判断、計画メモ、設計メモなどの目的・責務境界・受入条件・禁止事項が要件へ欠落なく落ちているかを検査する。design.md / tasks.md は参照資料であり、審査対象ではない
- design review：requirements.md → design.md を確認し、要件の目的・境界・受入条件が設計へ欠落なく落ちているかを検査する
- tasks review：requirements.md → design.md → tasks.md を確認し、要件と設計の意図が implementation-ready なタスク粒度へ落ちているかを検査する
- implementation review：requirements.md → design.md → tasks.md → implementation を確認し、実装がタスクだけでなく上流意図から逸脱していないかを検査する
```

(English gloss: for the requirements phase specifically, the review must check that the target's purpose/boundaries/acceptance-criteria/forbidden-actions are inherited from upstream decision materials without omission; `design.md`/`tasks.md` are reference-only, not judged.)

### 4.2 WORKFLOW_NAVIGATION.md#stage (full text of the cited section)

```
### `stage`

通常ワークフロー上の次タスクが決まっている。`feature`、`phase`、`stage` の示す作業だけを扱う。

`stage` が `triad-review` の場合、review-run の開始前に使用 variant と役ごとの provider／model を確定し、曖昧なまま開始しない。review-run に使うプロンプトは [[llm-as-judge-prompting]] の手順（材料揃え → 問い設計 → 選択肢なし分析）で設計する。API review-run 完了後は、proxy_model 判断依頼、実装修正、spec.json 更新、フェーズ移行のいずれにも進む前に、raw 参照、モデル別要約、同根所見クラスタ、`must-fix`／`should-fix`／`leave-as-is` の三段階トリアージ案、重要件の平易な説明を利用者へ提示して停止する。詳細手順は `.reviewcompass/guidance/SESSION_WORKFLOW_GUIDE.md#3.3-a-2` を正本とする。
```

(English gloss: for a triad-review stage, the variant and role assignments must be confirmed before starting; after the API review-run completes, results must be presented to the user — raw references, per-model summaries, same-root finding clusters, a three-level triage draft — before any proxy_model request, implementation edit, spec.json update, or phase transition.)

## 5. Review Target

- Path: `.reviewcompass/specs/term-representation/requirements.md`
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

1. 複数ケース間の汎化性能評価
2. 複数LLMの比較
3. 対照実験としてのGPのみ実行：最初に構築するLLMなしGPフレームワーク（後述）そのものを、LLM併用時との比較対象として使う
4. 収束履歴の可視化
5. 複数の評価地点：x/D = 2, 5, 10, 15。後流幅・中心速度欠損も評価の観点に加える
6. 既存モデルとのベンチマーク比較：Jensen、BP、Larsen、Frandsen

GPプロセス自体の精査点：時間方向の偏微分（d_dt演算子）は扱わない（対象は定常な代数式）。項（遺伝子）は、LLMSRと同様に内部に自由な係数（b, c, dなど）を持てる表現力の高い形式を維持する。項の自動選択と重要度の算出は、LLMSRから流用するDE（差分進化法）による係数最適化の評価関数に、不要な項の重みを0に近づける罰則（Lassoと同様の考え方）を加える方式を採用する。最適化後の各項の重みを正規化した値を、項の重要度として扱う。

## 対象データ

`data/`配下の風車後流CFDシミュレーション結果（座標・速度・圧力・乱流量を含むCSV）。今後、乱流強度I=0.01〜0.3とC=10〜25の組み合わせでケースを整備予定。
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

term-importance: 最適化後の項の重みを正規化し、重要度を算出する。依存: equation-fitting (hard)。
```

## 7. Required Checks

1. **purpose_transfer**: Does the target's overview (§1) correctly state that `term-representation` defines the GP gene/term data structure as the foundation for the other core features, consistent with intent's stated purpose? Failure condition: overview omits or misstates this foundational role.
2. **boundary_transfer**: Does the target's scope-out section (§4) correctly exclude crossover/mutation/seeding, DE coefficient optimization with L1 penalty, importance scoring, and CSV/case-parameter data preprocessing, matching the feature-partitioning's stated dependency boundaries? Failure condition: any of these responsibilities is implicitly claimed by the target's functional requirements, or a boundary is missing from the scope-out list.
3. **acceptance_criteria_transfer**: Do the target's functional requirements (FR1-FR5) operationalize intent's "GPプロセス自体の精査点" statement (expressive terms with free internal coefficients, no `d_dt`) into concrete, checkable acceptance criteria? Failure condition: FR2's operator list omits the `d_dt` exclusion, or FR3's free-coefficient requirement is missing or contradicts intent.
4. **forbidden_actions_transfer**: Does the target correctly forbid a `d_dt`/time-derivative operator and correctly avoid hard-coding the `load`-able variable list, per the user's explicit direction recorded in User Review Requirements above? Failure condition: `d_dt` appears as an allowed operator, or the variable list is presented as fixed/hard-coded rather than externally supplied.
5. **user_requirement_fidelity**: Are the three user-directed decisions in §3 (operator list, externally-supplied variable set including D/TI/C, and CSV/case-parameter preprocessing being out of scope) present in the target without narrowing or broadening? Failure condition: any of the three is missing, weakened, or altered.

## 8. Out of Scope

- The correctness or completeness of `design.md` or `tasks.md` for `term-representation` (neither exists yet).
- The correctness of `term-variation-operators`, `equation-fitting`, `evolution-loop-and-selection`, or `term-importance` beyond how `term-representation`'s requirements state its boundary with them.
- Implementation-level Julia design choices (data structure encoding, module layout); these are design-phase decisions.
- Whether the feature-partitioning split itself was the right one (already approved by the user).

## 9. Finding Policy

- Report `CRITICAL` if the target would authorize commit, push, phase completion, or human-only approval steps (it should not; this is a requirements document).
- Report `ERROR` if any of the 5 required checks in §7 fails (upstream purpose, boundary, acceptance criterion, or forbidden action is omitted, weakened, or contradicted).
- Report `ERROR` if the target adds a requirement not supported by any upstream source material (unsupported addition / drift).
- Report `WARN` for ambiguity, weak phrasing, or incomplete traceability that does not amount to an outright omission.
- Report `INFO` for minor clarity improvements.
- Return `findings: []` only if all 5 required checks pass and no unsupported additions are found.

## Output Contract

Return YAML only with top-level key `findings`. Each finding must include `severity` (CRITICAL/ERROR/WARN/INFO), `target_location` (specific section in `requirements.md`), `description`, and `rationale`. If there are no findings, return exactly `findings: []`.

