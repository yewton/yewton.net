# pagespeed-performance Specification

## Purpose
`https://www.yewton.net/` およびその配下ページ（HugoBlox でビルドされた静的サイト）
の PageSpeed・Lighthouse 計測に関する要件を定める。
HugoBlox 標準機能の範囲内で、Lighthouse Desktop Performance スコアと Core Web Vitals（LCP・CLS）
を良好判定に維持し、再訪問時の体感速度向上のためのキャッシュ設定を規定する。
## Requirements
### Requirement: Lighthouse Desktop で Performance スコア 95 以上を維持する

`https://www.yewton.net/` の Lighthouse Desktop 計測（preset=desktop、headless Chrome）における Performance カテゴリ スコアは SHALL 95 以上でなければならない。

#### Scenario: スコア計測値の取得
- **WHEN** パフォーマンス改善作業の検証フェーズに入ったとき
- **THEN** `npx -y lighthouse https://www.yewton.net/ --preset=desktop --only-categories=performance --chrome-flags="--headless --no-sandbox"` を実行し、Performance スコアを取得する
- **AND** スコアが 95 以上であることを確認する

### Requirement: ホームページの LCP と CLS を Core Web Vitals 良好判定に維持する

ホームページにおいて、Largest Contentful Paint は MUST 1.2 秒以下、Cumulative Layout Shift は MUST 0.05 以下でなければならない。

#### Scenario: LCP の計測
- **WHEN** Lighthouse 計測を実行したとき
- **THEN** `largest-contentful-paint` 監査の `numericValue` が 1200 ms 以下であることを確認する

#### Scenario: CLS の計測
- **WHEN** Lighthouse 計測を実行したとき
- **THEN** `cumulative-layout-shift` 監査の `numericValue` が 0.05 以下であることを確認する

### Requirement: ハッシュ付き不変アセットは 1 年キャッシュされる

Hugo ビルド時に SHA-256 ハッシュが付与される不変アセット（`_entry.<hash>.css`、`*.min.<hash>.js`、`/dist/font/*`、`/assets/js/preact-built/*`）は、HTTP `Cache-Control: public, max-age=31536000, immutable` で配信されなければならない (MUST)。

#### Scenario: ハッシュ付き CSS のキャッシュ検証
- **WHEN** `curl -I https://www.yewton.net/css/_entry.<hash>.css` を実行したとき
- **THEN** レスポンスの `Cache-Control` ヘッダーが `public, max-age=31536000, immutable` を含む

#### Scenario: ハッシュ付き JS のキャッシュ検証
- **WHEN** `curl -I https://www.yewton.net/js/hugo-blox-ja.min.<hash>.js` を実行したとき
- **THEN** レスポンスの `Cache-Control` ヘッダーが `public, max-age=31536000, immutable` を含む

### Requirement: 画像アセットは中期キャッシュされる

`/media/*` および `*.webp`・`*.jpg`・`*.png` 画像は、HTTP `Cache-Control: public, max-age=2592000` 以上で配信されなければならない (MUST)。

#### Scenario: メディア画像の中期キャッシュ検証
- **WHEN** `curl -I https://www.yewton.net/media/sharing.png` を実行したとき
- **THEN** レスポンスの `Cache-Control` ヘッダーが `max-age=2592000` 以上を含む

### Requirement: HTML は再検証のみキャッシュされる

ルートおよび配下の HTML は、HTTP `Cache-Control: public, max-age=0, must-revalidate` で配信されなければならない (MUST)。ブラウザが毎回コンテンツの新しさを確認することを許容する一方、不必要な長期キャッシュは行わない。

#### Scenario: HTML は再検証のみ
- **WHEN** `curl -I https://www.yewton.net/` を実行したとき
- **THEN** レスポンスの `Cache-Control` ヘッダーが `max-age=0, must-revalidate` を含む

### Requirement: ホームページの LCP 候補画像をプリロードする

ホームページにおいて、LCP 候補である hero 背景画像 `/media/home-bg.jpg` を `<link rel="preload" as="image" href="/media/home-bg.jpg" fetchpriority="high">` で HTML の `<head>` 内から先読みしなければならない (MUST)。LCP 候補が存在しないページ（記事ページ等）ではこのプリロード link タグを出力してはならない (MUST NOT)。

#### Scenario: ホームページでのプリロード出力
- **WHEN** ホームページの HTML ソースを確認したとき
- **THEN** `<link rel="preload" as="image" href="/media/home-bg.jpg" fetchpriority="high">` が `<head>` 内に存在する

#### Scenario: 記事ページでのプリロード非出力
- **WHEN** 任意の記事ページ（例: `/2026/08/22/minecraft-lan-connection-timed-out/`）の HTML ソースを確認したとき
- **THEN** `<link rel="preload" as="image" href="/media/home-bg.jpg" ...>` は存在しない

### Requirement: Inter フォントは既定で読み込まない

HugoBlox の font 設定（`site.Params.hugoblox.typography.font`）は `native` でなければならず (MUST)、Inter Variable woff2（325KB）を既定で読み込まない。これにより LCP 経路から Inter フォントのリクエストが排除される。

#### Scenario: Inter フォント参照の不存在
- **WHEN** ホームページの HTML ソースを確認したとき
- **THEN** `@font-face` ブロックに `Inter var` を含まない（あるいは `@font-face` ブロックそのものが存在しない）

#### Scenario: native 設定の永続化
- **WHEN** `config/_default/params.yaml` を確認したとき
- **THEN** `hugoblox.typography.font: native` が設定されている

