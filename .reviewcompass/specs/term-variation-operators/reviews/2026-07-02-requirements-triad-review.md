# term-variation-operators requirements triad-review（2026-07-02）

## 1. 概要

`term-variation-operators`のrequirements.mdに対する、上流意図伝達（vertical intent transfer）triad-review。variant: `implementation_review_independent_3way`（主役=claude-sonnet-4-6、敵対役=gpt-5.5、判定役=gemini-3.1-pro-preview）。

review-run記録：`.reviewcompass/specs/term-variation-operators/reviews/2026-07-02-term-variation-operators-requirements-review-run/`

## 2. モデル別結果

| 役 | model | 所見数 |
|---|---|---|
| primary | claude-sonnet-4-6 | 5（ERROR 2、WARN 2、INFO 1） |
| adversarial | gpt-5.5 | 2（ERROR 2） |
| judgment | gemini-3.1-pro-preview | 2（ERROR 2） |

## 3. 所見と対処

### F-001（must-fix、3モデル一致・クラスタ）

**内容**：FR1（初期集団の生成）の命令文モデルが、承認済みterm-representationの構造と矛盾。「最初の命令文は引数を持たないload」が誤り（loadは変数名を引数に持つ）。「2番目以降の命令文の引数は自分より前の命令文からのみ選ぶ」がterm-representation FR3（自由係数を直接引数にできる）を狭めていた。

**対処**：最初の命令文を「変数・ケースパラメータを指定するload」に修正。2番目以降の命令文の引数選択を「前の命令文の出力」または「自由係数のプレースホルダ」のいずれかに拡張。利用者承認済み（2026-07-02）。

### F-002（must-fix、2モデル一致：主役WARN＋判定役ERROR）

**内容**：FR2の「(k+1)番目以降」という表現が1始まりの数え方を前提にしており、添字非依存の方針に反する。

**対処**：「先頭k個を除いた残りの命令文」という序数を使わない表現に修正。利用者承認済み。

### F-003（must-fix、単独所見・primary）

**内容**：FR2の受け入れ基準が命令文数の一致のみを確認し、交叉の本質（プレフィックス交換）を検証していない。

**対処**：「子1の先頭k個は親2の先頭k個と一致、子2の先頭k個は親1の先頭k個と一致」という受け入れ基準を追加。利用者承認済み。

### F-004（must-fix、単独所見・primary）

**内容**：FR3の受け入れ基準「元の親項と少なくとも1つの命令文が異なる」が、アルゴリズムから保証されない過剰な制約（根拠のない追加）。

**対処**：「選んだ命令文が、FR1と同じ生成方法で作られた新しい命令文に置き換えられている」に変更。利用者承認済み。

### F-005（should-fix、単独所見・primary）

**内容**：FR3の「前に生成された」という表現が、生成順と並び順を混同しやすい。

**対処**：「前に位置する命令文」に修正。利用者承認済み。

### F-006（leave-as-is、単独所見・primary、INFO）

**内容**：FR4に確率既定値のdesign段階への繰り延べを直接明記すると分かりやすい。

**対処**：現状維持（§1・§4ですでに明記済みのため必須ではないと判断）。

## 4. 統合

F-001〜F-005を対処済み。F-006は利用者判断によりleave-as-is。修正後の`requirements.md`に、他の未解決所見はない。
