# タスク: 記事執筆ワークフローの Markdown ネイティブ化

## 1. ドキュメント更新
- [x] `CLAUDE.md` の "Content workflow (critical)" を書き換える。
  - [x] 「`content/` 直接編集禁止 / Org が正」の記述を撤廃し、Markdown を正とする新方式を記載。
  - [x] front matter テンプレート、Org→Markdown/ショートコード対応表を記載（`design.md` §3・§4 準拠）。
  - [x] 更新日ポリシー（`lastmod:` を意図的更新時のみ手動付与、`enableGitInfo` 不使用）を記載。
  - [x] 目次はテーマ自動生成に委ねる旨を記載。
  - [x] 既存記事・`content-org/*.org` は当面温存・参照専用である旨を明記。
- [x] `openspec/project.md` の "Tech Stack" と "Content Workflow" を現状（HugoBlox 移行後・Markdown ネイティブ）に刷新する。

## 2. 更新日表示の補正
- [x] `layouts/_partials/components/last-edited.html` をローカルオーバーライドとして追加し、
      `.Lastmod` が `.Date` より後のときだけ「最終更新」を描画するようにする（`design.md` §2）。
- [x] `config/_default/hugo.yaml` に `enableGitInfo` を追加しないことを確認する（現状未設定なので変更なし）。

## 3. 今回移行済み記事の後始末
- [x] `content/post/2026-08-22_minecraft-lan-connection-timed-out/index.md` の
      `<div class="ox-hugo-toc">` … `<!--endtoc-->` 静的目次ブロックを削除する。
- [x] `content-org/all-posts.org` の当該記事の Org 見出し（`** DRAFT Minecraft Java 版 …`）を削除する。

## 4. 検証
- [x] `hugo --gc --minify`（または `hugo server`）でビルドが通ることを確認する。
- [x] 対象記事で、テーマ自動生成の目次が表示され、削除した静的目次ブロックが消えていることを確認する。
- [x] `lastmod` を持たない記事で「最終更新 = 公開日」の重複行が出ないこと、
      `lastmod` を公開日より後に設定した記事でのみ「最終更新」が表示されることを確認する。
- [x] `{{< lastmod >}}` を使う既存 3 記事のビルドが壊れていないことを確認する。

## 5. レビュー反映（MiniMax-M3 レビュー＋ユーザー判断）
- [x] `layouts/shortcodes/lastmod.html` の `date_format` 参照を `site.Params.hugoblox.locale.date_format`（`| default ":date_long"`）へ修正し、インライン利用 2 記事の日付表示を確認。
- [x] 実更新あり（`mylastmod` > 公開日）・公開済み・ショートコード未使用の 10 記事で `mylastmod:` → `lastmod:` へ移し替え、テーマ「最終更新」に実更新日が出ることを確認。
- [x] Minecraft 記事の `mylastmod`（移行タイムスタンプ）を削除。
- [x] ビルドで「最終更新」を出す記事がちょうど 10 件（移し替え分）で、二重表示が無いことを確認。
