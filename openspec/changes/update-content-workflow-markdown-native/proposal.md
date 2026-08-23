# 提案: 記事執筆ワークフローの Emacs 非依存化（Markdown ネイティブ化）

## 概要

ブログ記事の執筆・公開ワークフローを、Emacs Org-mode + `ox-hugo` を単一ソースとする方式から、
Hugo ページバンドル配下の Markdown（`content/post/<bundle>/index.md`）を直接編集・執筆する
Hugo / HugoBlox 標準の方式へ移行します。

目的は次の 2 点です。

- **Emacs 依存の排除**: Org → Emacs 再エクスポートという human-in-the-loop の手順をなくす。
- **エージェントとの共著の容易化**: Emacs を持たない外部レビューエージェントも含め、
  執筆・レビュー・修正を Markdown 上で完結できるようにする。

本提案は**今後の新規記事の執筆方式の確立**に範囲を限定します。既存記事および
`content-org/*.org` は当面そのまま温存し、参照専用とします（Org 依存の全面撤廃は将来の別変更）。

## スコープ

- **ドキュメントの更新**
  - `CLAUDE.md` の "Content workflow (critical)" を Markdown ネイティブ方式へ書き換える。
    「`content/` 直接編集禁止」の記述を撤廃し、新方式のルール（front matter テンプレート、
    ショートコード対応、更新日ポリシー、目次の扱い）を記載する。
  - `openspec/project.md` の "Tech Stack" / "Content Workflow" を現状（HugoBlox 移行後）に合わせて
    刷新する（現状は Wowchemy / Emacs 前提の古い記述が残存）。
- **更新日（最終更新）ポリシーの確立**
  - 新規記事では、Hugo 標準の front matter `lastmod:` を、**内容を意図的に更新したときだけ手動で付与**
    する運用に統一する。機械的・軽微な修正では更新日を動かさない。
  - `enableGitInfo` は**有効化しない**（git コミット日時から自動導出すると機械的修正でも更新日が
    動いてしまい、上記ポリシーと矛盾するため）。
  - テーマの「最終更新」表示（`last-edited.html`）が、未更新記事でも公開日を「最終更新」として
    重複表示する挙動を、`lastmod` が公開日より後のときだけ表示するようローカルオーバーライドで補正する。
- **目次（TOC）方針の明文化**
  - 目次は HugoBlox が `.Headings` から自動生成する仕組み（右サイドバー／`{{< toc >}}` ショートコード）に
    委ねる。新規記事では静的な目次ブロックを手書きしない。
- **今回移行した記事の後始末**
  - `content/post/2026-08-22_minecraft-lan-connection-timed-out/index.md` に残る ox-hugo 由来の
    静的目次ブロック（`<div class="ox-hugo-toc">` … `<!--endtoc-->`）を削除する。
  - `content-org/all-posts.org` 内の当該記事の Org 見出し（`** DRAFT Minecraft Java 版 …`）を削除し、
    Markdown 側（公開済み）との状態の矛盾を解消する。

## スコープ外

- 既存記事の一括変換（Org 記法の残骸整理、`{{% callout %}}` → Markdown alert 変換など）。
- `content-org/*.org` 全体の廃止・アーカイブ、および `ox-hugo` 関連設定の撤去。
- 既存の `{{< lastmod >}}` ショートコードおよび `mylastmod` front matter の削除
  （既存記事が参照しているため互換目的で残す。新規記事では使わない、と定めるにとどめる）。
- 記事内容そのものの加筆・修正。

## 影響

- **執筆プロセス**: 今後の新規記事は Org を経由せず Markdown を直接編集する。
- **`CLAUDE.md` / `openspec/project.md`**: ワークフロー記述が新方式に置き換わる。
- **`config/_default/hugo.yaml`**: 目次の自動表示に関わる設定（既存の `tableOfContents` はそのまま）。
  `enableGitInfo` は追加しない。
- **`layouts/_partials/components/last-edited.html`**: ローカルオーバーライドを追加（新規ファイル）。
  既存記事・新規記事の双方に作用するが、`lastmod == date` の記事では従来出ていた重複行が消えるのみで、
  ビルドは壊れない。
- **既存記事**: `{{< lastmod >}}` ショートコード（3 記事が使用）は互換のため残すので、既存記事の
  ビルド・表示は壊れない。

## 追記（レビュー反映）

MiniMax-M3 によるレビューとユーザー判断を受け、当初スコープ外としていた次の項目を追加で実施した。

1. **`{{< lastmod >}}` ショートコードのバグ修正**: 参照先を旧 Wowchemy パス
   `site.Params.locale.date_format`（移行後は未解決で日付書式が崩れていた）から
   `site.Params.hugoblox.locale.date_format`（`| default ":date_long"`）へ修正。互換で残す
   インライン利用 2 記事が正しい日本語日付で表示されることを確認。
2. **公開後に実更新した記事の更新日復元**: `mylastmod` を独自キーとして持つため HugoBlox テーマの
   「最終更新」に反映されていなかった記事のうち、`mylastmod` が公開日より後（実更新あり）で、
   公開済み・本文ショートコード未使用の 10 記事について、front matter の `mylastmod:` を標準
   `lastmod:` へ移し替え、テーマの「最終更新」に実更新日が表示されるようにした。
3. **新規記事の `mylastmod` 除去**: 今回移行した Minecraft 記事の `mylastmod`（移行時タイムスタンプで
   実内容更新ではない）を削除。新方式では標準 `lastmod` のみを用いる方針に合わせた。
