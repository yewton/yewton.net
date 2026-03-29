# HugoBlox (Tailwind CSS) への移行調査報告

HugoBlox を Bootstrap ベース (v5.9.7) から最新の Tailwind CSS ベース (`github.com/HugoBlox/kit/modules/blox`) へ移行するための初期調査、およびトップページの修正を実施しました。

## 現状と実施した変更

以下の設定を更新し、最新版でのビルドが通る状態まで調整しました。
- `go.mod`: モジュール参照を `blox-bootstrap` から `blox`へ変更。
- `config/_default/hugo.yaml`: モジュールインポートのパスを更新。
- `.tool-versions` / `netlify.toml`: Hugo バージョンを `0.159.1` へ更新。
- **注意**: ビルドには `tailwindcss` CLI と `@tailwindcss/typography` などの依存パッケージが必要です。これらを管理するため、リポジトリに `package.json` と `package-lock.json` を追加しました。
- **トップページの修正**: `content/home/index.md` を `type = "widget_page"` から最新の `blocks` (blox) 形式へ書き換えました。これに伴い、`hero.md` と `posts.md` は削除し、`index.md` に統合しました。

## 判明している課題 (今後のステップ)

### 1. 他の Widget Page の移行 (Blog, CV)
トップページと同様に、`content/blog/index.md` や `content/cv/index.md` も `widget_page` を使用しているため、最新の `blocks` 形式への書き換えが必要です。
- **Blog**: `content/blog/posts.md` や `tags.md` を `index.md` の `blocks` に統合する必要があります。
- **CV**: `experience.md` や `skills.md` などを統合し、適切な `blox` 名を指定する必要があります。

### 2. デザイン・レイアウトの再調整
Bootstrap から Tailwind CSS へフレームワークが刷新されたため、以前のスタイル定義がほぼ無効化されています。
- **症状**: ブログ記事一覧の余白異常、ナビゲーションバーのレイアウト崩れ、サイドバーの重なりなど。
- **対応**: 各ページのフロントマターにおける `design` パラメータの再調整、および `assets/` 内のカスタム CSS の Tailwind 準拠への書き換えが必要です。

### 3. 著者プロフィール (Authors) のデータ構造変更
`content/authors/admin/_index.md` などの著者の情報が、最新版では `data/authors/` 以下で管理されるデータ駆動型システムへの移行が推奨されています。
- **症状**: 記事末尾のプロフィール表示や SNS アイコンが正しく表示されません。
- **対応**: 著者の情報を `data/authors/` 形式へ移動し、各記事からの参照方法を更新する必要があります。

### 4. 非推奨となったショートコード
`callout` ショートコードなどの一部が非推奨となり、標準の Markdown Alert 構文 (`> [!NOTE]` 等) への移行が求められています。

### 5. Org-mode ソースへの反映
今回の修正は `content/*.md` に対して直接実施されました。プロジェクトの運用ルールに従い、これらの変更を `content-org/*.org` 側にも反映させる必要があります。
- **対象**: `content-org/site.org` の `Home` セクションなど。
