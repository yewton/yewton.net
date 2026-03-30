---
title: "Wowchemy から HugoBlox (Tailwind CSS 版) への移行記"
author: ["yewton"]
date: 2026-03-30T00:00:00+09:00
mylastmod: 2026-03-30T00:00:00+09:00
slug: "wowchemy-to-hugoblox"
tags: ["Hugo", "HugoBlox", "Wowchemy", "Tailwind CSS", "Netlify"]
categories: ["技術系"]
draft: false
---

<div class="ox-hugo-toc toc">

<div class="heading">目次</div>

- [はじめに](#はじめに)
- [移行の背景](#移行の背景)
- [問題1: トップページが真っ白になった](#問題1-トップページが真っ白になった)
- [問題2: デザインが崩れた (Bootstrap → Tailwind CSS)](#問題2-デザインが崩れた)
- [問題3: 著者プロフィールのデータ構造変更](#問題3-著者プロフィールのデータ構造変更)
- [問題4: CV ページが空になった](#問題4-cv-ページが空になった)
- [問題5: callout ショートコードの廃止](#問題5-callout-ショートコードの廃止)
- [問題6: params.yaml の名前空間変更](#問題6-paramsyaml-の名前空間変更)
- [問題7: Netlify でビルドが失敗した](#問題7-netlify-でビルドが失敗した)
- [問題8: CSS が大幅に欠落した (hugo_stats.json)](#問題8-css-が大幅に欠落した)
- [まとめ](#まとめ)

</div>
<!--endtoc-->


## はじめに {#はじめに}

このサイトはずっと [Hugo](https://gohugo.io/) + [Wowchemy](https://wowchemy.com/)（現 HugoBlox）で動いていました。
いつの間にか Wowchemy は HugoBlox にリブランドし、テーマエンジンも Bootstrap から Tailwind CSS へと刷新されていました。

しばらく放置していましたが、重い腰を上げて最新の HugoBlox (Tailwind CSS 版) へ移行することにしました。
この記事では、移行中に踏んだ地雷とその対処法をまとめます。


## 移行の背景 {#移行の背景}

旧システムは `go.mod` で `github.com/HugoBlox/hugo-blox-builder/modules/blox-bootstrap` を参照していました。
新システムは `github.com/HugoBlox/kit/modules/blox` に移っており、内部的には Bootstrap が完全に廃止され Tailwind CSS v4 に置き換えられています。

Hugo のバージョンも `0.159.1` まで上げる必要がありました（`.tool-versions` と `netlify.toml` を更新）。

さらに、新システムのビルドには `tailwindcss` CLI と `@tailwindcss/typography` などの npm パッケージが必要になるため、`package.json` と `package-lock.json` をリポジトリに追加しました。


## 問題1: トップページが真っ白になった {#問題1-トップページが真っ白になった}

最初にして最大の衝撃でした。ビルドは通るのに、トップページが完全に空白です。

**原因:** 旧システムの `type = "widget_page"` 方式が完全に廃止されていました。
`content/home/` 配下のウィジェットファイル群（`hero.md`、`posts.md` 等）は新システムでは一切読まれません。

**新しい方式:** `content/_index.md` に `type: landing` を指定し、`sections:` リストでブロックを定義します。

```yaml
---
title: yewton.net
type: landing

sections:
  - block: hero
    content:
      title: yewton.net
      # hero の text は Markdown リンクを解釈しないので HTML で書く
      text: 'yewton が <a href="/blog/" class="underline">ブログ</a> を書いたり ...'
    design:
      no_padding: true
      css_class: dark
      css_style: "background-image: linear-gradient(rgba(0,0,0,0.5), rgba(0,0,0,0.5)), url('/media/home-bg.jpg'); background-size: cover; background-position: top;"
      spacing:
        padding: ['80px', '0', '80px', '0']

  - block: collection
    id: posts
    content:
      title: 最近の投稿
      count: 5
      order: desc
      filters:
        folders:
          - post
    design:
      view: card
---
```

**移行作業:** `content/_index.md` を新規作成し、旧 `content/home/` 配下のファイルをすべて削除しました。
Hero ブロックが Preact ベースなので `preact` npm パッケージの追加も必要でした。

**得られた知見をいくつか:**

- **Hero ブロックは Preact ベース**: `block.html` をカスタムオーバーライドしても効果がありません。クライアントサイドで Preact がレンダリングするためです。
- **Hero の `text` は Markdown リンク未対応**: `[text](url)` 形式のリンクは解釈されません。リンクを含む場合は HTML で直接書く必要があります。
- **背景画像へのダーク overlay**: `design.background.image.filename` 経由で設定すると CSS の順序の関係でグラデーションが背景に隠れてしまいます。代わりに `design.css_style` で `linear-gradient(rgba(...), rgba(...)), url(...)` と直接書きましょう。その場合、画像は `static/media/` に置いて固定 URL `/media/filename.jpg` で参照します。
- **投稿一覧のビュー**: 旧 `view = 2`（compact）に相当するものはありません。`card` か `date-title-summary` が近い選択肢です。


## 問題2: デザインが崩れた {#問題2-デザインが崩れた}

Bootstrap から Tailwind CSS へフレームワークが刷新されたため、以前の `assets/scss/custom.scss` のスタイルがほぼ無効化されました。

**原因:** 新システムは `assets/css/custom.css` のみ読み込みます。`assets/scss/custom.scss` は完全に無視されます。

**対応内容:**

- `assets/css/custom.css` を新規作成。
- SCSS のネスト記法をフラットな CSS に変換。
- Tailwind で再実装済みのスタイル（`#profile .age` / `#anniversary-list` 等）は削除。
- `.src-block-caption`・`.featured-image-credit`・`.ox-hugo-toc`・`pre.chroma` 等の引き続き必要なスタイルは維持。


## 問題3: 著者プロフィールのデータ構造変更 {#問題3-著者プロフィールのデータ構造変更}

`resume-biography`・`resume-experience`・`resume-skills` 等のブロックを使おうとしたら、何も表示されませんでした。

**原因:** 著者データの管理場所が変わっていました。

| 旧 | 新 |
|---|---|
| `content/authors/admin/_index.md` | `data/authors/admin.yaml` |

新システムのブロックはすべて `data/authors/{slug}.yaml` を参照します。
`content/authors/` は参照しません。

**その他の変更点:**

- **アバター画像の格納先**: 旧 `content/authors/admin/avatar.png` → 新 `assets/media/authors/admin.jpg`
- **リンクのアイコン名**: 旧 `icon_pack: fab` + `icon: github` → 新 `icon: brands/github`（パック名をスラッシュで連結）
- **職歴の日付キー名**: 旧 `date_start` / `date_end` → 新 `start` / `end`

スキルデータは以下の形式です:

```yaml
skills:
  - name: カテゴリ名
    items:
      - label: Ruby
        icon: devicon/ruby   # devicon パックのアイコン名（766種）
        level: 4             # 1〜5（バープログレスとして表示）
        description: "**業務5年** | 説明テキスト"
```


## 問題4: CV ページが空になった {#問題4-cv-ページが空になった}

トップページと同様に、`/cv/` 以下も `type = "widget_page"` を使っていたため、新システムでは何も表示されない状態でした。

**対応内容:** `content/cv/_index.md` を `type: landing` + `sections:` 形式で再構築。

| 旧ファイル | 旧ウィジェット | 新ブロック |
|---|---|---|
| `about.md` | `about-ext`（カスタム） | `resume-biography` |
| `experience.md` | `experience` | `resume-experience` |
| `pl_skills.md` 他 | `featurette-l`（カスタム） | `resume-skills` |
| `topics.md` | `blank` | `markdown` |
| `accomplishments.md` | `accomplishments` | `resume-awards` |

なお、紹介されていた `tech-stack` ブロックは現バージョンには含まれていないため、`resume-skills` ブロックで代替しました。


## 問題5: callout ショートコードの廃止 {#問題5-callout-ショートコードの廃止}

過去記事で使っていた `{{</* callout note */>}}` が壊れました。

**対応:** Markdown Alert 構文へ一括置換しました。

| 旧 | 新 |
|---|---|
| `{{</* callout note */>}}` / `{{</* callout info */>}}` | `> [!NOTE]` |
| `{{</* callout warning */>}}` | `> [!WARNING]` |

9ファイルを置換しました。


## 問題6: params.yaml の名前空間変更 {#問題6-paramsyaml-の名前空間変更}

ビルドは通ってサイトも表示されるのに、Disqus コメント欄・日付フォーマット・フッター著作権・Google Analytics がすべて機能していませんでした。

**原因:** `params.yaml` の構造が変わっていました。旧 Wowchemy は `features.*`・`locale.*`・`marketing.*` という名前空間でしたが、新 HugoBlox は `hugoblox.*` しか読みません。

また、旧 `config.toml` にあったブログ投稿向けカスケード設定（`commentable`・`show_related`・`show_breadcrumb`）が `hugo.yaml` に移行されておらず、コメント欄・関連記事・パンくずリストが非表示になっていました。

**対応内容:**

`params.yaml` を `hugoblox:` ネームスペースに全面書き換え:

```yaml
hugoblox:
  locale:
    date_format: "2006年1月2日"
    time_format: "15:04"
  copyright:
    notice: "© {year} yewton"
  comments:
    provider: disqus
    disqus:
      shortname: yewton
  analytics:
    google:
      measurement_id: "G-XXXXXXXXXX"
  layout:
    avatar_shape: circle
```

`hugo.yaml` にカスケード設定を追加:

```yaml
cascade:
  - target:
      path: /post/**
    commentable: true
    show_related: true
    show_breadcrumb: true
```

> [!NOTE]
> Hugo v0.156.0 で `cascade._target` が非推奨になり `cascade.target` に変更されました。アンダースコアなしです。

`reading_time` と `share` は新システムでデフォルト ON のため、cascade 設定は不要です。


## 問題7: Netlify でビルドが失敗した {#問題7-netlify-でビルドが失敗した}

ローカルでは動くのに、Netlify では `ERROR failed to load modules: "_vendor/..." not found` が発生してビルドが失敗しました。

**原因:** グローバル gitignore（`~/.config/git/ignore`）に `*.com` というルールがあり、`_vendor/github.com/` ディレクトリ全体が Git に追跡されていませんでした。`_vendor/modules.txt` だけコミットされていたため、Hugo が「ベンダリングあり」と判断してローカルにない `_vendor/github.com/` を必須とした結果、Netlify でコケていました。

**対応:** ベンダリングをやめ、ビルド時にモジュールをダウンロードする方式に変更しました。`_vendor/` を削除し、`/_vendor` を `.gitignore` に追加。Netlify の環境には Go が使えるので問題なく動作します。

開発者固有のグローバル gitignore ルールがプロジェクトファイルの追跡を妨げる、という典型的な罠でした。


## 問題8: CSS が大幅に欠落した {#問題8-css-が大幅に欠落した}

ベンダリング廃止後、サイトの見た目が大幅に崩れました。CSS ファイルは生成されているのに、約 40KB 分のスタイルが欠落していました。

**原因:** HugoBlox の Tailwind CSS v4 は、Hugo が生成する `hugo_stats.json` をクラス検出ソースとして利用します（`@source "hugo_stats.json"`）。この仕組みは次の流れで機能します：

1. Hugo がすべてのテンプレートを処理しながら `hugo_stats.json` をディスクへ書き出す
2. `templates.Defer` で遅延実行される `css.html` が `hugo_stats.json` を参照して Tailwind を実行
3. HugoBlox テンプレート内のすべてのクラスが CSS に含まれる

`hugo_stats.json` が生成されない場合、Tailwind はユーザーの `layouts/` と `content/` のみをスキャンし、HugoBlox テンプレート側のクラスを検出できなくなります。

**なぜベンダリングありのときは動いていたか:** `_vendor/` が存在する環境では、HugoBlox モジュールの `hugo.yaml` に書かれた `build.buildStats.enable: true` がプロジェクト設定にマージされていました。ベンダリングを廃止したときにこの設定がマージされなくなったため、`hugo_stats.json` が生成されなくなっていました。

**対応:** `config/_default/hugo.yaml` に明示的に追加するだけです:

```yaml
build:
  buildStats:
    enable: true
```

`hugo_stats.json` はビルド生成物なので `.gitignore` に追加してコミット対象外にします。

CSS サイズが極端に小さい（本サイトの場合、正常時 186KB のところ 146KB 程度）場合はこの問題を疑ってください。Hugo モジュールの `hugo.yaml` 設定は、すべてがプロジェクト設定にマージされるとは限りません。特に `build.*` 系の設定は、プロジェクト側に明示的に記述する必要がある場合があります。


## まとめ {#まとめ}

踏んだ地雷を整理すると以下のとおりです。

| # | 問題 | 一言まとめ |
|---|---|---|
| 1 | トップページが空白 | `widget_page` 廃止、`type: landing` + `sections:` 方式へ |
| 2 | デザイン崩れ | `scss/custom.scss` が無視される、`css/custom.css` を使う |
| 3 | 著者データが表示されない | `content/authors/` → `data/authors/` へ |
| 4 | CV ページが空 | トップページと同様に再構築 |
| 5 | callout ショートコード廃止 | Markdown Alert 構文（`> [!NOTE]`）へ置換 |
| 6 | コメント欄・Analytics が動かない | `params.yaml` を `hugoblox:` 名前空間に書き直す |
| 7 | Netlify でビルド失敗 | グローバル gitignore の罠、ベンダリング廃止で解決 |
| 8 | CSS が欠落 | `hugo_stats.json` 生成に `build.buildStats.enable: true` が必要 |

全体を通じて「古いドキュメントがそのまま残っている」「モジュール側の設定がサイレントにマージされない」という Hugo / HugoBlox 特有の罠が多い印象でした。何か動かないときはまず HugoBlox のモジュールキャッシュ（`~/.cache/hugo_cache/modules/`）を直接覗いて、現在の実装を確認するのが近道です。

同じ移行を検討している方の参考になれば幸いです。
