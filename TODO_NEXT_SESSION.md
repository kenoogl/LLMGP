# LLMGP - 次セッション継続用メモ

最終更新：2026-07-02。正本は `tools/check-workflow-action.py next --json` と各 feature の `spec.json`。この TODO は入口メモであり、作業順序の正本ではない。

作業ディレクトリ：`/Users/Daily/Development/WindTurbineWake/LLMGP/`
リポジトリ：`git@github.com:kenoogl/LLMGP.git`（main ブランチ）
ReviewCompass 配布物：`/Users/Daily/Development/WindTurbineWake/ReviewCompass/`（絶対パスで参照。LLMGP内には配置しない）

## 0. 全体像

LLMGPは、隣接プロジェクトLLMSR（風車後流のCFDデータからLLM併用の疑似GPで代数式モデルを探索するJulia製研究基盤）の作り直し。まずLLMを使わない自動処理だけでGPフレームワーク（項の表現・交叉変異・DEによる係数最適化・項の重要度算出）を完成させ、一通りの機能を確認してからLLMを導入する段階的開発方針。詳細は`intent/INTENT.md`・`intent/NON_GOALS.md`・`intent/DESIGN_PRINCIPLES.md`を参照。

feature一覧・依存関係は`.reviewcompass/feature-dependency.yaml`（`stages/feature-partitioning/2026-07-02-proposal.md`で承認済み）。今回確定させたfeature_orderは5つ：`term-representation` → `term-variation-operators` → `equation-fitting` → `evolution-loop-and-selection` → `term-importance`。intentに記載の6つの追加機能（複数ケース評価・複数LLM比較・収束可視化・複数評価地点・ベンチマーク比較・LLM導入）は、必要になった時点で個別にfeature-partitioningし直す。

## 1. 起動時に必ず行うこと

1. `/usr/bin/python3 /Users/Daily/Development/WindTurbineWake/ReviewCompass/tools/check-workflow-action.py next --json` を実行し、`next_action`を作業順序の正本として扱う。
2. `git status --short`と`git log --oneline -8`で到達点を確認する。
3. `next_action.required_disciplines`に出た規律だけを、操作直前に読む。

API経由のreview-runでAPIキーを使う場合、このセッションのBashツールは`.zshrc`を自動読み込みしないため、`source ~/.zshrc && <コマンド>`の形で実行する（詳細は記憶`feedback_commit_no_verify.md`等を参照）。

## 2. 不可逆操作の規律

- spec.jsonのworkflow_state変更、commit、push、規律ファイル変更は利用者の明示承認が必要。
- **重要**：`.claude/settings.json`のPreToolUse hook登録は無効化済み（ReviewCompass側`commit-preflight`の`load_all_feature_specs`が自身のFEATURE_ORDERを参照してしまうバグのため）。通常の`git commit`をそのまま使ってよい。ReviewCompass側でバグが直ったら復元を検討する。
- triad-reviewを実施する場合は、実行前にvariantと役ごとのprovider/modelを利用者へ提示する。操縦LLMがClaude系（claude-sonnet-5等）のため、prompt quality review用variantは既定の`api_review_prompt_quality_2way`ではなく、`api_review_prompt_quality_2way_openai_adversarial`（敵対役をgpt-5.5に変更、ReviewCompass配布物側の`config/api-settings.yaml`に追加済み）を使う。3役triad-reviewは`implementation_review_independent_3way`（primary=claude-sonnet-4-6／adversarial=gpt-5.5／judgment=gemini-3.1-pro-preview）。
- triad-review実施時は、必ず「①main preanalysis→②preanalysis sufficiency audit（判定役が問題なしを返すまで繰り返す、完了後に利用者へ簡潔な要約報告）→③criteria起草→④mainによる厳格なセルフチェック（Claude Codeのサブエージェントによる一次チェックを使ってよい。正式な独立監査の代替ではない）→⑤prompt quality review→⑥実review-run→⑦raw/parsed/summary/triage→⑧必要ならproxy_model判断→⑨レビュー結果の簡潔な要約報告」の9段階を踏む（`API_REVIEW_PROMPT_QUALITY.md`の規律、2026-07-02にequation-fittingレビューを機に④・要約報告の2点を追記済み）。criteria草案・監査用バンドルを作る際は、複数文書を結合するときのコードフェンス（```）の入れ子崩れに注意し、送信前にサブエージェントでの事前検査を挟むと効率的（term-variation-operators・equation-fittingの両レビューで有効だった）。prompt quality reviewは1回で収束しないことがある（equation-fittingでは5ラウンド要した）。両役（adversarial・judgment）がfindings: []を返すまで、指摘を反映して再実行を繰り返す。

## 3. 現在位置

- feature: `evolution-loop-and-selection`
- phase: `requirements`
- stage: `drafting`
- reason: `evolution-loop-and-selection の requirements.drafting が未完了です`

## 4. 次作業

`evolution-loop-and-selection`のrequirements drafting。intentの「世代交代のループ本体」を、`term-variation-operators`のrequirements.md（子の生成：交叉・変異）と`equation-fitting`のrequirements.md（方程式の組み立てと評価：係数最適化）を踏まえて要件化する。責務は「子の生成→方程式の組み立てと評価→エリート戦略による選択、を繰り返す」（`stages/feature-partitioning/2026-07-02-proposal.md`）。2N個（親N＋子N）のプールからエリート選択でN個へ絞る具体的な選択方式は、intentやfeature-partitioningに明記がないため、要件化時に確認・利用者判断が必要になる可能性がある。

drafting完了後は、§2に記載の9段階手順でtriad-reviewを実施する。

## 5. 直近の完了事項

- ReviewCompass初期設定一式、intent 3文書の起草・承認
- feature-partitioning承認、`.reviewcompass/feature-dependency.yaml`作成
- `term-representation`：requirements drafting・triad-review完了（所見2件対処）
- `term-variation-operators`：requirements drafting・triad-review完了（所見6件対処、古賀2018修論の交叉・変異アルゴリズムを反映）
- `equation-fitting`：requirements drafting・triad-review完了（起草時に「項の出力を正規化してから重みを掛ける」要件を利用者指示で追加。triad-reviewで所見1件対処：LLMSR評価関数の記述誤り「加減算・乗算」→「加算・乗算」。prompt quality reviewは5ラウンドで収束）。この過程で`API_REVIEW_PROMPT_QUALITY.md`にセルフチェック段・繰り返し規律・要約報告の追記を実施（配布物側、gitなし、§2参照）

## 6. 参照

- workflow navigation: `.reviewcompass/guidance/WORKFLOW_NAVIGATION.md`（ReviewCompass配布物側）
- session guide: `.reviewcompass/guidance/SESSION_WORKFLOW_GUIDE.md`（ReviewCompass配布物側）
- prompt quality: `.reviewcompass/guidance/API_REVIEW_PROMPT_QUALITY.md`（ReviewCompass配布物側）
- disciplines: `.reviewcompass/guidance/discipline_*.md`（ReviewCompass配布物側）
- spec.json: `.reviewcompass/specs/<feature>/spec.json`
- intent: `intent/INTENT.md`、`intent/NON_GOALS.md`、`intent/DESIGN_PRINCIPLES.md`
- feature分割: `stages/feature-partitioning/2026-07-02-proposal.md`
