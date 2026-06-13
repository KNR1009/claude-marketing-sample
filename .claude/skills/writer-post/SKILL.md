---
name: writer-post
description: X投稿の下書きを作成する。「投稿を作って」「Xに書きたい」で起動
---

# 手順
1. data/atoms.csv からステータス「未使用」のネタを読む
2. 1ネタにつき投稿案を2パターン作成（140字以内 / 全角換算）
3. 構成: フック1行 → 本文 → 問いかけで締め
4. 出力先: drafts/YYYY-MM-DD-post.md
5. 使用したネタの atoms.csv のステータスを「下書き済」に更新