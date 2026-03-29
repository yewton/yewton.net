# HugoBlox (Tailwind CSS) への移行調査報告

HugoBlox を Bootstrap ベース (v5.9.7) から最新の Tailwind CSS ベース (`github.com/HugoBlox/kit/modules/blox`) へ移行するための調査・作業を実施しました。

## 現状と実施した変更

以下の設定を更新し、最新版でのビルドが通る状態まで調整しました。
- `go.mod`: モジュール参照を `blox-bootstrap` から `blox` へ変更。
- `config/_default/hugo.yaml`: モジュールインポートのパスを更新。
- `.tool-versions` / `netlify.toml`: Hugo バージョンを `0.159.1` へ更新。
- **注意**: ビルドには `tailwindcss` CLI と `@tailwindcss/typography` などの依存パッケージが必要です。これらを管理するため、リポジトリに `package.json` と `package-lock.json` を追加しました。

## 判明した主な問題点と対応状況

### 1. トップページが空白になる (Widget Page の非推奨) ✅ 対応済み

従来の `type = "widget_page"` を用いたトップページ構成が、最新の Tailwind 版では正しくレンダリングされません。

**原因と仕組み:**
新システムは `content/_index.md` に `type: landing` を指定し、frontmatter の `sections:` リストでブロックを定義する方式に変わりました。旧来の `content/home/*.md` ウィジェットファイルは完全に廃止されています。

**対応内容:**
- `content/_index.md` を新規作成し、`type: landing` + `sections:` 形式で記述。
- 旧 `content/home/` 配下のファイル（`index.md`、`hero.md`、`posts.md`）を削除。
- `preact` npm パッケージを追加（hero ブロックが Preact ベースのため必要）。

**設定例 (`content/_index.md`):**
```yaml
---
title: yewton.net
type: landing

sections:
  - block: hero
    content:
      title: yewton.net
      # hero の text フィールドは Markdown リンクを解釈しない（後述）ため HTML で記述
      text: 'yewton が <a href="/blog/" class="underline">ブログ</a> を書いたり ...'
    design:
      no_padding: true
      css_class: dark
      # 背景画像のダーク overlay は css_style で実現（後述）
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

**得られた知見:**

- **Hero ブロックは Preact ベース**: `blox/hero/component.jsx` がクライアントサイドでレンダリングされるため、サーバーサイドの `block.html` はフォールバックとしてのみ機能する。`block.html` のカスタムオーバーライドは Preact ブロックには効果がない。

- **Hero の text フィールドは Markdown リンク未対応**: `component.jsx` の `renderText()` 関数は `**bold**`・`*italic*`・ `` `code` `` のみ処理し、`[text](url)` 形式のリンクは解釈しない。リンクを含む場合は HTML で直接記述する必要がある。

- **背景画像のダーク overlay**: `parse_block_v3` の実装上、`design.background.image.filename` を使うと CSS が `background-image: url(...), linear-gradient(...)` の順（画像が前面）になり、グラデーションがオーバーレイとして機能しない。代わりに `design.css_style` で `linear-gradient(rgba(...), rgba(...)), url(...)` と記述することで、グラデーションを前面に配置してダーク overlay を実現できる。この場合、画像は `assets/media/` ではなく `static/media/` に置き、ビルドに依存しない固定 URL `/media/filename.jpg` で参照する。

- **利用可能なブロック一覧**: `_vendor/github.com/HugoBlox/kit/modules/blox/blox/` 配下を参照。現時点では `collection`、`hero`、`markdown`、`features`、`cta-card` 等が利用可能。

- **背景設定の正しいフォーマット** (`design.background`):
  - グラデーション: `gradient.start` / `gradient.end`（旧: `gradient_start` / `gradient_end`）
  - 背景画像: `image.filename`（旧: `image` 文字列）
  - テキスト色: `text_color_light: true` でセクションに `dark` クラスが付与される

- **投稿一覧のビュー**: 旧 `view = 2`（compact）に相当するビューは新システムには存在しない。最も近い選択肢は `card`（モダンな縦型カード）または `date-title-summary`（サムネイルなしシンプルリスト）。

### 2. デザイン・レイアウトの崩れ 🔄 部分対応済み

Bootstrap から Tailwind CSS へフレームワークが刷新されたため、以前のスタイル定義がほぼ無効化されています。
- **症状**: ブログ記事一覧の余白異常、ナビゲーションバーのレイアウト崩れ、サイドバーの重なりなど。
- **修正内容**: 各ページのフロントマターにおける `design` パラメータの再調整、および `assets/` 内のカスタム CSS の Tailwind 準拠への書き換えが必要です。

### 3. 著者プロフィール (Authors) のデータ構造変更 ⬜ 未対応

`content/authors/admin/_index.md` などの著者の情報が、最新版では `data/authors/` 以下で管理されるデータ駆動型システムへの移行が推奨されています。
- **症状**: 記事末尾のプロフィール表示や SNS アイコンが正しく表示されません。
- **修正内容**: 著者の情報を `data/authors/` 形式へ移動し、各記事からの参照方法を更新する必要があります。

### 4. CV（オープン職務経歴書）ページの再構築 ⬜ 未対応

`/cv/` 以下はトップページと同様に `type = "widget_page"` を用いた構成であり、新システムでは何も表示されない状態です。

**現在の構成（`content/cv/`）とウィジェット対応表:**

| ファイル | 旧ウィジェット | 内容 | 新ブロック候補 |
|---|---|---|---|
| `about.md` | `about-ext`（カスタム） | 著者プロフィール「人となり」 | `resume-biography` / `resume-biography-3` |
| `experience.md` | `experience` | 職務要約（タイムライン形式） | `resume-experience` |
| `pl_skills.md` | `featurette-l`（カスタム） | プログラミング言語スキル表 | `resume-skills` / `markdown` |
| `mw_skills.md` | `featurette-l`（カスタム） | ミドルウェアスキル表 | `resume-skills` / `markdown` |
| `cloud_skills.md` | `featurette-l`（カスタム） | クラウドスキル表 | `resume-skills` / `markdown` |
| `topics.md` | `blank` | やってきたこと（自由記述） | `markdown` |
| `accomplishments.md` | `accomplishments` | 資格など | `resume-awards` |

**課題:**
- `about-ext` と `featurette-l` はもともとカスタムウィジェット（`layouts/partials/blocks/v1/` に残存）であり、新システムの blox 形式への書き直しが必要。
- `experience` ウィジェットは TOML の `[[experience]]` ブロック配列でデータを持つが、新システムの `resume-experience` ブロックはデータ形式が異なる可能性がある。
- `cv/index.md` を `type: landing` + `sections:` 形式に書き換え、各 `.md` ファイルのコンテンツを `sections:` のインラインデータとして統合する必要がある。

**参考情報:**
Developer Portfolio テーマが resume 系ブロックの実装例として最も参考になる。

- デモ: https://hugo-theme-developer-portfolio.netlify.app/
- ソース: https://github.com/HugoBlox/hugo-theme-developer-portfolio/blob/main/content/_index.md

上記テーマの `_index.md` は `resume-biography`、`resume-experience`、`resume-skills` 等のブロックを実際に組み合わせており、設定値の書き方やデータ構造の参考になる。

### 5. 非推奨となったショートコード ⬜ 未対応

`callout` ショートコードなどの一部が非推奨となり、標準の Markdown Alert 構文 (`> [!NOTE]` 等) への移行が求められています。
ビルド時に対象ファイルが警告として列挙されるため、それを手がかりに順次対応できます。

## 今後のステップ

- [ ] CV ページの `type: landing` + `sections:` 形式への再構築（問題 4）
- [ ] 著者プロフィールの `data/authors/` 形式への移行（問題 3）
- [ ] `callout` ショートコードの Markdown Alert 構文への置き換え（問題 5）
- [ ] カスタム CSS の Tailwind 準拠への書き換え（問題 2 の残作業）
