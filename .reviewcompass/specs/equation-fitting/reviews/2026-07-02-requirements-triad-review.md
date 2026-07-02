# equation-fitting requirements triad-review（2026-07-02）

## 1. 概要

`equation-fitting`のrequirements.mdに対する、上流意図伝達（vertical intent transfer）triad-review。variant: `implementation_review_independent_3way`（主役=claude-sonnet-4-6、敵対役=gpt-5.5、判定役=gemini-3.1-pro-preview）。

prompt quality review（`api_review_prompt_quality_2way_openai_adversarial`）は5ラウンド実施し、両役ともfindings: []に収束させてから実review-runへ進んだ。

review-run記録：`.reviewcompass/specs/equation-fitting/reviews/2026-07-02-equation-fitting-requirements-review-run/`

## 2. モデル別結果

| 役 | model | 所見数 |
|---|---|---|
| primary | claude-sonnet-4-6 | 6（WARN 1、INFO 5） |
| adversarial | gpt-5.5 | 1（ERROR 1） |
| judgment | gemini-3.1-pro-preview | 0 |

## 3. 所見と対処

### F-001（must-fix、単独所見・adversarial、ERROR）

**内容**：§3非機能要件・制約で、LLMSRの既存評価関数を「物理妥当性の罰則を加減算・乗算する形式」と記述しているが、grounding evidence（今回のセッションで実施したLLMSR調査結果）と照合すると「減算」の形式は存在しない。実際はPhase5が加算式（`mse + physical_penalty`）、Phase6が乗算式（`mse * (1.0 + penalty)`）のみ。

**対処**：「加減算・乗算する形式」を「加算・乗算する形式」に訂正した。利用者承認済み（2026-07-02）。

### F-002（should-fix、単独所見・primary、WARN）

**内容**：FR1の正規化に関する受け入れ基準は、「その時点の項内部係数の値に基づいて」正規化が追従することは明記しているが、正規化が同じ入力データ（データセット）に対して評価されるという前提は明記されていない。係数依存性のみを述べており、データ依存性が暗黙のままになっている。

**対処**：明示的な追加指示がなかったため、現状維持（design段階での具体化に委ねる）。

### F-003〜F-007（leave-as-is、primary、INFO 5件）

**内容**：概要でのFR1定義への正規化組み込み、FR3のペナルティ強度0での還元性、非機能要件でのLLMSR参考値の非拘束的な明示、スコープ外境界の網羅性、LLMSR評価関数（MSEのみ、正則化なし）の記述正確性——いずれも「正しく反映されている」という肯定的確認。対処不要。

## 4. 統合

F-001を対処済み。F-002は利用者判断により現状維持。F-003〜F-007は対処不要。修正後の`requirements.md`に、他の未解決所見はない。
