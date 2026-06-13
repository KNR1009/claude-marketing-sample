---
name: research-ai
description: Claude/Gemini/OpenAI系の直近のアップデート情報を収集し、一次情報で裏取りして data/atoms.csv に追記する。「AIニュース集めて」「直近◯日のアップデート調べて」「ネタ収集」で起動する。
---

# research-ai — AIニュース収集スキル

## いつ使うか
ユーザーが「AIニュースを集めて」「直近3日のClaude/Gemini/OpenAIのアップデートを調べて」「記事のネタを集めて」と言ったとき。

## 前提
- 開始前に必ず CLAUDE.md の「品質基準」と「データレイヤー」を読むこと。
- 既存の `data/atoms.csv` を読み込み、重複（同じ url または同じ title）を追加しないこと。

## 手順

1. 収集対象を確認する
   - 対象ベンダー: Anthropic(Claude) / Google(Gemini) / OpenAI（指定があればそれに従う）
   - 期間: 指定がなければ「今日から遡って3日間」

2. Web検索・公式ソースで情報を集める
   - 優先順位: ①公式ブログ/リリースノート ②公式X/発表 ③信頼できる技術メディア
   - 一次情報URL（公式）を必ず1件は確保する。二次情報のみのネタは summary 末尾に「（要一次裏取り）」と書く

3. 各ニュースを1行のatomに整形する
   - id: `a` + 連番（既存atoms.csvの最大id+1）
   - date: 発表日(YYYY-MM-DD)
   - source: 取得元メディア名 or 公式
   - vendor: anthropic / google / openai
   - title: 簡潔な見出し（誇張語を使わない）
   - url: 一次情報のURL
   - summary: 80〜120字。何が・どう変わったかを実務目線で
   - status: new

4. `data/atoms.csv` に追記する（既存行は消さない・上書きしない）

5. 収集結果をユーザーに要約報告する
   - 「◯件追加（Claude ◯/Gemini ◯/OpenAI ◯）」と内訳を出す
   - 一次裏取りが取れていないものがあれば明示する

## 品質チェック（このスキルのOutput Gate）
- [ ] 各atomに発表日とURLが入っている
- [ ] 誇張語（神/ヤバい/最強 等）を使っていない
- [ ] 期間外（指定日より古い）のニュースを混ぜていない
- [ ] 既存atomsと重複していない

## 出力先
- `data/atoms.csv`（追記）