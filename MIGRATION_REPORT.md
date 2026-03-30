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

- **利用可能なブロック一覧**: Hugo モジュールキャッシュ（`~/.cache/hugo_cache/modules/filecache/modules/pkg/mod/github.com/!hugo!blox/kit/modules/blox@vX.Y.Z/`）配下を参照。現時点では `collection`、`hero`、`markdown`、`features`、`cta-card`、`resume-biography`、`resume-experience`、`resume-skills`、`resume-awards` 等が利用可能。

- **背景設定の正しいフォーマット** (`design.background`):
  - グラデーション: `gradient.start` / `gradient.end`（旧: `gradient_start` / `gradient_end`）
  - 背景画像: `image.filename`（旧: `image` 文字列）
  - テキスト色: `text_color_light: true` でセクションに `dark` クラスが付与される

- **投稿一覧のビュー**: 旧 `view = 2`（compact）に相当するビューは新システムには存在しない。最も近い選択肢は `card`（モダンな縦型カード）または `date-title-summary`（サムネイルなしシンプルリスト）。

### 2. デザイン・レイアウトの崩れ ✅ 対応済み

Bootstrap から Tailwind CSS へフレームワークが刷新されたため、以前のスタイル定義がほぼ無効化されています。

**対応内容:**
- 新システムは `assets/css/custom.css` のみ読み込む（`assets/scss/custom.scss` は無視される）ため、`assets/css/custom.css` を新規作成。
- SCSS のネスト記法をフラットな CSS に変換。
- 新テンプレートで Tailwind に置き換えた `#profile .age` / `#anniversary-list` スタイルは削除。
- `.src-block-caption`・`.featured-image-credit`・`.ox-hugo-toc`・`.mylastmod`・`pre.chroma` 等の引き続き必要なスタイルは維持。

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
  - `icon` には `devicon/` プレフィックスで devicon（766種）が使用可能。利用可能な名前は Hugo モジュールキャッシュ内の `data/icons/devicon.json` のキーを参照。
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

### 5. 非推奨となったショートコード ✅ 対応済み

`callout` ショートコードなどの一部が非推奨となり、標準の Markdown Alert 構文 (`> [!NOTE]` 等) への移行が求められています。

**対応内容:**
9ファイルの `callout` ショートコードを Markdown Alert 構文へ一括置換。
- `note` / `info` → `> [!NOTE]`
- `warning` → `> [!WARNING]`

## 今後のステップ

- [x] CV ページの `type: landing` + `sections:` 形式への再構築（問題 4）
- [x] 著者プロフィールの `data/authors/` 形式への移行（問題 3）
- [x] `callout` ショートコードの Markdown Alert 構文への置き換え（問題 5）
- [x] カスタム CSS の Tailwind 準拠への書き換え（問題 2 の残作業）

---

## 移行後確認結果（master との差分調査）

master（旧 Wowchemy v5）と移行版（HugoBlox Tailwind）を全体比較した結果、以下の差異を確認した。

### 確認済み・問題なし

- **設定ファイル (`config/`)**: TOML → YAML 変換済み。`defaultContentLanguage: ja`・`hasCJKLanguage: true` は `hugo.yaml` に移行済み。旧 `languages.toml` の `languageCode = "ja-jp"` は、単一言語サイトでは実質的な影響なし（HTML `lang` 属性は `ja` になるが問題ない）。
- **著者プロフィール CSS (`#profile .age` / `#anniversary-list`)**: 旧 SCSS に存在したスタイルだが、新しいカスタム `resume-biography/block.html` では完全に Tailwind CSS で再実装済みであるため不要。
- **メニュー (`menus.yaml`)**: 全項目が移行済み。
- **静的アセット (`static/`)**: ファビコン・画像等はすべて移行済み（`static/media/` は移行版で新たに追加）。
- **ショートコード (`layouts/shortcodes/`)**: `lastmod`・`pexels`・`pixabay`・`unsplash` の 4 つがそのまま維持されている。
- **ブログ記事コンテンツ (`content/post/`)**: 全記事が移行済み。

### 残課題

#### 6. ブログページのタグクラウドが消えた ✅ 廃止（対応済み）

旧 `content/blog/tags.md`（"Popular Topics" タグクラウドウィジェット）は移行時に削除済み。新 HugoBlox に相当する組み込みブロックが存在しないため、廃止とする。タクソノミーページ `/tags/` は引き続き存在する。

#### 7. ブログ投稿のカスケード設定・params.yaml の構造変更 ✅ 対応済み

**発覚した問題:**
- 旧 `params.yaml` は Wowchemy 形式（`features.*`・`locale.*`・`marketing.*`）だったが、新 HugoBlox は `site.Params.hugoblox.*` しか参照しない。そのため Disqus コメント・日付フォーマット・フッター著作権・Google Analytics がすべて機能していなかった。
- 旧 `config.toml` にあったブログ投稿向けカスケード設定（`commentable`・`show_related`・`show_breadcrumb`）が `hugo.yaml` に移行されておらず、コメント欄・関連記事・パンくずリストが非表示だった。

**対応内容:**
- `params.yaml` を `hugoblox:` ネームスペースに全面書き換え:
  - `hugoblox.locale.date_format` / `time_format`（日本語日付フォーマット）
  - `hugoblox.copyright.notice`（フッター著作権、`{year}` 変数対応）
  - `hugoblox.comments.provider: disqus` + `disqus.shortname: yewton`（Disqus コメント）
  - `hugoblox.identity.type: Person` + `identity.social.twitter: yewton`（SEO）
  - `hugoblox.analytics.google.measurement_id: "G-SJ7FG3D0T4"`（GA4）
  - `hugoblox.layout.avatar_shape: circle`
  - `hugoblox.content.math.enable: false`
- `hugo.yaml` に cascade セクションを追加（`/post/**` 配下に適用）:
  - `commentable: true`・`show_related: true`・`show_breadcrumb: true`

**得られた知見:**
- `reading_time` と `share` は新システムでデフォルト ON（`ne .Params.x false` で判定）のため cascade 設定不要。
- Google Analytics は `analytics` サブモジュールが `blox` モジュール依存として既に含まれており、`hugoblox.analytics.google.measurement_id` を設定するだけで有効になる。
- Hugo v0.156.0 で `cascade._target` が非推奨になり `cascade.target` に変更された。

#### 8. Netlify ビルドエラーと Hugo モジュール管理 ✅ 対応済み

**発覚した問題:**

Netlify で `ERROR failed to load modules: "_vendor/..." not found` が発生してビルドが失敗した。

**原因:**

`_vendor/` ディレクトリをベンダリングする構成になっていたが、グローバル gitignore（`~/.config/git/ignore`）に `*.com` というルールがあり、`_vendor/github.com/` ディレクトリ全体が Git に追跡されていなかった。`_vendor/modules.txt` のみコミットされていたため、Hugo が「ベンダリングあり」と判断してローカルにない `_vendor/github.com/` を必須とした。

**対応内容:**

ベンダリングをやめ、ビルド時にモジュールをダウンロードする方式に変更。
- `_vendor/` を削除し、`_vendor/modules.txt` の Git 追跡を解除。
- `/_vendor` を `.gitignore` に追加。
- Netlify ビルド環境には Go と hugo が mise でインストールされるため、モジュールのダウンロードは問題なく動作する。

**得られた知見:**

- **Hugo モジュールのベンダリング方針**: `_vendor/` をコミットする（再現性重視・ネットワーク不要）か、しない（リポジトリ軽量・ビルド時ダウンロード）かのトレードオフがある。Netlify のようにビルド環境に Go が使える場合はベンダリングなしで十分。
- **グローバル gitignore の罠**: 開発者固有のグローバル gitignore ルール（`*.com` 等）がプロジェクトファイルの追跡を妨げる場合がある。CI/CD 環境ではこのようなローカル設定が存在しないため、ローカルでは動いても Netlify でコケる原因となる。

#### 9. Tailwind CSS の CSS 生成と `hugo_stats.json` ✅ 対応済み

**発覚した問題:**

ベンダリング廃止後、サイトの見た目が大幅に崩れた。CSS ファイルは生成されていたが、約 40KB 分のスタイルが欠落していた。

**原因:**

HugoBlox の Tailwind CSS v4 は、Hugo が生成する `hugo_stats.json` をクラス検出ソースとして利用する（`@source "hugo_stats.json"`）。この仕組みは次の流れで機能する：

1. Hugo がすべてのテンプレートを処理しながら `hugo_stats.json` をディスクへ書き出す
2. `templates.Defer` で遅延実行される `css.html` が `hugo_stats.json` を参照して Tailwind を実行
3. HugoBlox テンプレート内のすべてのクラスが CSS に含まれる

`hugo_stats.json` が生成されない場合、Tailwind はユーザーの `layouts/` と `content/` のみをスキャンし、Hugo の仮想ファイルシステム経由でマウントされた HugoBlox テンプレート側のクラスを検出できなくなる。

**なぜベンダリングありのときは動いていたか:**

`_vendor/` が存在するローカル環境では、HugoBlox モジュールの `hugo.yaml`（`build.buildStats.enable: true`）が正しくマージされ、`hugo_stats.json` が生成・保持されていた。ベンダリングを廃止してモジュールをダウンロード方式に切り替えた際、このモジュール設定がプロジェクト設定にマージされなかったため `hugo_stats.json` が生成されなくなった。

**対応内容:**

`config/_default/hugo.yaml` に明示的に追加：

```yaml
build:
  buildStats:
    enable: true
```

`hugo_stats.json` はビルド生成物のため `/.gitignore` に追加し、コミット対象外とした。

**得られた知見:**

- **Hugo モジュールの設定マージ**: モジュールの `hugo.yaml` に書かれた設定がすべてプロジェクト設定にマージされるとは限らない。特に `build.*` 系の設定は、プロジェクトの `hugo.yaml` に明示的に記述する必要がある場合がある。
- **`hugo_stats.json` のライフサイクル**: ビルドごとに再生成されるが、`templates.Defer` の遅延実行により「ページ処理完了 → `hugo_stats.json` 書き出し → CSS 生成」の順序が保証されるため、初回ビルドから正しく動作する。コミット不要。
- **CSS サイズの目安**: `hugo_stats.json` あり・なしで生成される CSS サイズが大きく異なる（本サイトでは約 146KB → 186KB）。CSS が極端に小さい場合はこの問題を疑うとよい。
