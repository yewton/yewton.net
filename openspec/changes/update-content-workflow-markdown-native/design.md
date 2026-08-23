# 設計メモ: Markdown ネイティブ化に伴う技術判断

会話とは独立して読まれることを前提に、Emacs / ox-hugo が肩代わりしていた処理を Hugo ネイティブへ
置き換える際の判断を記録します。調査は HugoBlox モジュール `github.com/HugoBlox/kit/modules/blox@v0.11.0`
（ビルド時取得、`_vendor` なし）の実体を参照して行いました。

## 1. 目次（TOC）

- **判断**: 目次リストは手書きせず、HugoBlox の自動生成に委ねる。ただし自動サイドバーは
  **デスクトップのみ**表示されるため、モバイル向けには本文先頭に `{{< toc >}}` ショートコードを
  記事単位で手動追加する（手書きの目次リストではなく、テーマの自動生成に委ねる点は同じ）。
- **根拠**: モジュールの `layouts/_partials/components/toc.html` が `.Headings` から目次を生成し、
  記事テンプレート `layouts/single.html` がデスクトップ右サイドバー（`xl:block`）として描画する。
  `.Params.toc`（既定 `true`）で切り替え可能。モバイルでは既定で目次が出ないため、
  `layouts/_shortcodes/toc.html`（`.Page.TableOfContents` を `<details … xl:hidden>` で描画、
  デスクトップでは非表示）を `{{< toc >}}` として本文先頭に置き、サイドバーと重複させずに補う。
- **アンカー ID の整合**: 目次はレンダリング後の実際の見出し ID を用いるため、ox-hugo が付けていた
  `## 見出し {#…}` の明示アンカーは不要。素の Markdown 見出しで ID ずれは起きない。

## 2. 更新日（最終更新）

### 背景となる要求
`mylastmod` は元々、**内容を意図的に更新したときだけ**手動で付与するフィールドだった。
機械的・軽微な修正で更新日が動くのを避けるための運用である。

### テーマの既定挙動（調査結果）
- 記事テンプレート `layouts/single.html` は、上部に公開日 `.Date`、末尾に
  `components/last-edited.html` を**無条件で**描画する。
- `last-edited.html` は `show_date_updated`（既定 `true`）かつ `.Lastmod` が真なら「最終更新」を表示する。
  Hugo の `.Lastmod` は front matter の `lastmod:` 未指定時に `date` へフォールバックするため、
  **未更新記事でも「最終更新 = 公開日」が重複表示される**。
- プロジェクト独自の `{{< lastmod >}}` ショートコード（`layouts/shortcodes/lastmod.html`）は
  `mylastmod` を読む。参照していた `site.Params.locale.date_format` は HugoBlox 移行で
  `hugoblox:` 配下へ移動しており未解決だった（日付が Go 既定フォーマットで崩れていた）。
  レビュー指摘を受け、`site.Params.hugoblox.locale.date_format`（`| default ":date_long"`）へ
  修正済み。互換のため残す 3 記事のインライン表示が正しい日本語日付で描画されることを実機確認した。

### 判断
- **新規記事の運用**: Hugo 標準の front matter `lastmod:`（ISO 8601、例 `2026-08-25T11:00:00+09:00`）を
  **意図的な内容更新時のみ**手動付与する。`mylastmod` と `{{< lastmod >}}` は新規記事では使わない。
- **`enableGitInfo` は有効化しない**: git コミット日時から `.Lastmod` を導出すると、移行に伴う機械的
  修正やタイポ修正でも更新日が動き、上記要求と矛盾する。
- **表示の補正（ローカルオーバーライド）**: `layouts/_partials/components/last-edited.html` を
  プロジェクト側に用意し、`.Lastmod` が `.Date` より**厳密に後**のときだけ「最終更新」を描画する。
  これにより「最終更新 = 公開日」の重複表示が消え、`lastmod:` を意図的に付与した記事だけ表示される。

  ```
  {{- if and (.Params.show_date_updated | default true) (.Lastmod.After .Date) -}}
    … 最終更新の描画 …
  {{- end -}}
  ```

- **既存記事への影響**: このオーバーライドは全記事に作用するが、`lastmod == date` の既存記事では
  従来の重複行が消えるだけでビルドは壊れない。`{{< lastmod >}}` を本文に持つ 3 記事は、
  そのインライン表示を従来どおり維持する（ショートコードは残す）。

### 代替案（不採用）
- テーマ標準の `show_date_updated` を `/post` の cascade で既定 `false` にし、更新時のみ記事側で
  `show_date_updated: true` と `lastmod:` を併記する案。ローカルオーバーライドを持たずに済むが、
  更新のたびにフィールドを 2 つ書く必要があり、`lastmod:` 1 つで完結する採用案より手間が増えるため見送り。

## 3. Org マクロ／記法 → Hugo での書き方

新規記事は素の Markdown で書く。対応関係は執筆ルールとして `CLAUDE.md` に記載する。

| Org（もう使わない） | Markdown / Hugo での書き方 |
|---|---|
| `~code~` / `=text=` | `` `code` ``（1 種類） |
| `*bold*` | `**bold**`（Org の「強調直後に全角句読点を置けない」制約は解消） |
| `[[url][text]]` | `[text](url)` |
| `#+begin_src lang` … `#+end_src` | ` ```lang ` フェンス |
| `#+begin_example` … `#+end_example` | ` ```text ` フェンス |
| `#+begin_quote` … `#+end_quote` | `>` 引用 |
| `{{{unsplash(...)}}}` 等の Org マクロ | 対応する Hugo ショートコードを直接記述（`{{< unsplash … >}}` 等） |
| `{{{begin_callout(note)}}}` | Markdown alert（`> [!NOTE]` / `> [!WARNING]`） |
| `{{{toc}}}` / `#+toc:` | 記述不要（テーマが自動生成、§1） |

ショートコード実体（`layouts/shortcodes/` の `unsplash` / `pexels` / `pixabay` / `lastmod`）は現存し、
`{{< name … >}}` 構文でそのまま呼べる。

## 4. front matter テンプレート（新規記事）

```yaml
---
title: "記事タイトル"
author: ["yewton"]
date: 2026-08-25
# lastmod: 2026-08-30T11:00:00+09:00   # 内容を意図的に更新したときだけ追記する
slug: "url-slug"
tags: ["tag1", "tag2"]
categories: ["雑記"]   # 既存カテゴリ（例: 技術系）から選ぶ
draft: true            # 公開時に false（DONE 化は人間が判断）
image:
  caption: 画像のクレジット/キャプション
---
```

記事は `content/post/<YYYY-MM-DD_slug>/index.md` にページバンドルとして置き、画像を同ディレクトリに並べる。

## 5. 今回移行済み記事の後始末

- `content/post/2026-08-22_minecraft-lan-connection-timed-out/index.md` の `<div class="ox-hugo-toc">`
  … `<!--endtoc-->` ブロックを削除（テーマ自動目次と重複するため）。見出しの明示アンカー `{#…}` は
  無害なので今回は残置可（削除は任意）。
- `content-org/all-posts.org` の当該 Org 見出し（DRAFT 状態のまま残存）を削除。Markdown 側は既に
  `draft: false` で公開済みであり、状態の矛盾を解消する。
