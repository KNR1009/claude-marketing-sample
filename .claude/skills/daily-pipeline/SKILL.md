---
name: daily-pipeline
description: AIニュースの「収集→記事→X投稿→レビュー」を一気通貫で回す司令塔スキル。research-ai / writer-article / writer-post / review を順に実行し、成果物ファイルを揃える。「今日のパイプライン回して」「daily-pipeline」「一連の流れを自動で」で起動する。
---

# daily-pipeline — 司令塔（オーケストレーター）スキル

## 役割（重要）
このスキルは **順番を回すだけの司令塔** です。収集・執筆・投稿・レビューの具体的な手順は
各スキル（research-ai / writer-article / writer-post / review）のSKILL.mdが唯一の情報源です。
ここにロジックを書き直さないこと。各ステップでは対応スキルの手順をそのまま呼び出す。

## いつ使うか
「今日のパイプラインを回して」「一連の流れを自動でやって」「daily-pipeline」と言われたとき。

## 前提
- 開始前に必ず CLAUDE.md の「品質基準(Output Gate)」「承認ルール」「データレイヤー」を読む。

## パラメータ（冒頭でユーザーに確認、未指定なら既定値）
- 対象ベンダー: 既定 = anthropic, google, openai
- 期間: 既定 = 今日から遡って3日間
- 記事の想定文字数: 既定 = 2000〜3000字
- 1回で出す記事本数: 既定 = 1本（複数atomをまとめる）

## 実行フロー

### STEP 0: 重複ガード（冪等性）
1. `data/outputs.csv` と `data/pipeline.csv` を読む。
2. 今日の日付で既に `note` の記事が `drafted` 以降になっている場合は、二重生成しない。
   ユーザーに「本日分は既に存在します。再生成しますか？」と確認し、Noなら中断。

### STEP 1: 収集（research-ai を実行）
1. research-ai の手順に従い、対象ベンダー・期間のニュースを収集し `data/atoms.csv` に追記。
2. **チェック**: 新規atomが1件以上増えたか確認。
   - 0件なら「収集対象が見つからなかった」と報告して **中断**（先に進まない）。
   - 一次裏取りが取れていないatomがあれば一覧で報告し、続行可否をユーザーに確認。

### STEP 2: 記事執筆（writer-article を実行）
1. writer-article の手順に従い、STEP1で増えた status=new のatomから記事を執筆。
2. 出力: `drafts/YYYY-MM-DD-note-<スラッグ>.md`
3. 使ったatomを status=picked に更新、`pipeline.csv` に channel=note, status=drafted を追記。
4. **チェック**: 記事冒頭の本数宣言（「N本」「N つ」）と本文の見出し数が一致しているか。
   不一致なら writer-article に差し戻して修正（FIX 1 の再発防止）。

### STEP 3: X投稿（writer-post を実行）
1. writer-post の手順に従い、STEP2の記事から告知ポスト＋主要atomの単体ポストをA/B作成。
2. 出力: `drafts/YYYY-MM-DD-x-posts.md`
3. `pipeline.csv` に channel=x, status=drafted を追記。
4. **チェック**: 各案が140字以内か。超過があれば writer-post に差し戻し。

### STEP 4: レビュー（review を実行）= Output Gate
1. review の手順に従い、記事とX投稿を5観点（事実/出典/帰属/誇張/重複）で判定。
2. 総合判定で分岐:
   - **PASS** → pipeline該当行を status=reviewed に更新し、STEP 5へ。
   - **FIX** → 指摘内容を該当スキル（writer-article / writer-post）に渡して修正させ、STEP 4を再実行。
   - **BLOCK** → 出典不足等。修正してもPASSできない場合は **中断してユーザーに報告**。
3. 自動修正ループは最大2回まで。3回目もPASSしなければ中断して人間に委ねる。

### STEP 5: 完了報告（公開はしない）
1. 生成された成果物を一覧で提示:
   - `drafts/YYYY-MM-DD-note-<スラッグ>.md`
   - `drafts/YYYY-MM-DD-x-posts.md`
2. review の総合判定（PASS）と、残る注意点があれば併記。
3. **実投稿・公開はしない**（承認ルール）。outputs.csv への記録用1行を提示し、
   「公開はご自身で行い、公開後にこの行を outputs.csv に追記してください」と案内。

## 中断条件（黙って進ませない）
- STEP1で新規atom 0件
- 一次裏取りが全く取れない
- review が BLOCK、または修正2回でPASSに到達しない
これらは必ずユーザーに状況を報告して停止する。

## このスキルがやらないこと
- 各スキルのロジックの再実装（子スキルに委譲）
- X/noteへの実投稿、メール送信などの外部公開操作

## 関連
- 子スキル: research-ai, writer-article, writer-post, review
- 共通ルール: CLAUDE.md
- データ: data/atoms.csv, data/pipeline.csv, data/outputs.csv