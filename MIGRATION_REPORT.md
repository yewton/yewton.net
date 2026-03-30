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

- **利用可能なブロック一覧**: `_vendor/github.com/HugoBlox/kit/modules/blox/blox/` 配下を参照。現時点では `collection`、`hero`、`markdown`、`features`、`cta-card`、`resume-biography`、`resume-experience`、`resume-skills`、`resume-awards` 等が利用可能。

- **背景設定の正しいフォーマット** (`design.background`):
  - グラデーション: `gradient.start` / `gradient.end`（旧: `gradient_start` / `gradient_end`）
  - 背景画像: `image.filename`（旧: `image` 文字列）
  - テキスト色: `text_color_light: true` でセクションに `dark` クラスが付与される

- **投稿一覧のビュー**: 旧 `view = 2`（compact）に相当するビューは新システムには存在しない。最も近い選択肢は `card`（モダンな縦型カード）または `date-title-summary`（サムネイルなしシンプルリスト）。

### 2. デザイン・レイアウトの崩れ 🔄 部分対応済み

Bootstrap から Tailwind CSS へフレームワークが刷新されたため、以前のスタイル定義がほぼ無効化されています。
- **症状**: ブログ記事一覧の余白異常、ナビゲーションバーのレイアウト崩れ、サイドバーの重なりなど。
- **修正内容**: 各ページのフロントマターにおける `design` パラメータの再調整、および `assets/` 内のカスタム CSS の Tailwind 準拠への書き換えが必要です。

### 3. 著者プロフィール (Authors) のデータ構造変更 ✅ 対応済み

旧システムでは `content/authors/admin/_index.md` で著者情報を管理していたが、新システムは `data/authors/{slug}.yaml` のデータ駆動型へ移行。

**対応内容:**
- `data/authors/admin.yaml` を新規作成。スキーマは `hugoblox/author/v1`。
- 旧 `content/authors/admin/_index.md` および旧カスタムウィジェット (`layouts/partials/blocks/v1/`) を削除。

**得られた知見:**

- **著者プロフィールの参照先**: `resume-biography`・`resume-experience`・`resume-skills`・`resume-awards` 等のブロックはすべて `data/authors/{slug}.yaml` を参照する（`content/authors/` は参照しない）。Hugo の `get_author_profile` partial が `site.Data.authors` を読む仕組み。

- **アバター画像の格納先変更**: 旧システムは `content/authors/admin/avatar.png` に置いていたが、新システムは `assets/media/authors/{slug}.*`（例: `assets/media/authors/admin.jpg`）に置く必要がある。`get_author_profile` が `resources.GetMatch "media/authors/{slug}.*"` でアセットを解決する。

- **`links` のアイコン名の変更**: 旧: `icon_pack: fab` + `icon: github` → 新: `icon: brands/github`（パック名をプレフィックスとしてスラッシュで連結する形式）。

- **`experience` のキー名変更**: 旧: `date_start` / `date_end` → 新: `start` / `end`（`resume-experience` ブロックはどちらも受け付けるが、新スキーマに準拠するなら `start`/`end` を推奨）。

- **`resume-skills` のスキルデータ形式**:
  ```yaml
  skills:
    - name: カテゴリ名
      description: 補足テキスト（省略可）
      items:
        - label: Ruby
          icon: devicon/ruby   # devicon パックのアイコン名
          level: 4             # 1〜5（バープログレスとして表示）
          description: "**業務5年** | 説明テキスト（Markdown 可）"
  ```
  - `icon` には `devicon/` プレフィックスで devicon（766種）が使用可能。利用可能な名前は `_vendor/.../data/icons/devicon.json` のキーを参照。
  - `level` は 1〜5 の整数で習熟度バーとして描画される。

### 4. CV（オープン職務経歴書）ページの再構築 ✅ 対応済み

`/cv/` 以下はトップページと同様に `type = "widget_page"` を用いた構成であり、新システムでは何も表示されない状態でした。

**対応内容:**
`content/cv/_index.md` を `type: landing` + `sections:` 形式で再構築。旧ウィジェットファイル群は削除。

| 旧ファイル | 旧ウィジェット | 新ブロック |
|---|---|---|
| `about.md` | `about-ext`（カスタム） | `resume-biography`（`content.text` で本文をインライン記述） |
| `experience.md` | `experience` | `resume-experience`（`data/authors/admin.yaml` の `experience` を参照） |
| `pl_skills.md` / `mw_skills.md` / `cloud_skills.md` | `featurette-l`（カスタム） | `resume-skills`（`data/authors/admin.yaml` の `skills` を参照） |
| `topics.md` | `blank` | `markdown` |
| `accomplishments.md` | `accomplishments` | `resume-awards`（`data/authors/admin.yaml` の `awards` を参照） |

**得られた知見:**

- **`resume-biography` の本文**: ブロックの `content.text` に Markdown を直接書くことで `data/authors/{slug}.yaml` の `bio` フィールドを上書きできる。CV ページ専用の詳細な自己紹介文を別途持ちたい場合に有効。

- **`resume-skills` のカテゴリ単位での説明文**: `design.columns` で列数（1〜5）を制御できる。カテゴリの `description` フィールドに注釈（「※2020年1月時点」等）を入れると各カテゴリ見出しの下に表示される。ブロック全体の補足はブロックの `content.text` に書くとよい。

- **`resume-experience` の日付フォーマット**: `design.date_format` に Go の時刻レイアウト文字列（例: `"2006年1月"`）を指定可能。

- **`tech-stack` ブロックは v0.11.0 には未収録**: 利用例として示されている `tech-stack` ブロックはこのバージョンに含まれていない。devicon アイコン付きのスキル表示は `resume-skills` ブロックで代替できる。

### 5. 非推奨となったショートコード ⬜ 未対応

`callout` ショートコードなどの一部が非推奨となり、標準の Markdown Alert 構文 (`> [!NOTE]` 等) への移行が求められています。
ビルド時に対象ファイルが警告として列挙されるため、それを手がかりに順次対応できます。

## 今後のステップ

- [x] CV ページの `type: landing` + `sections:` 形式への再構築（問題 4）
- [x] 著者プロフィールの `data/authors/` 形式への移行（問題 3）
- [ ] `callout` ショートコードの Markdown Alert 構文への置き換え（問題 5）
- [ ] カスタム CSS の Tailwind 準拠への書き換え（問題 2 の残作業）
