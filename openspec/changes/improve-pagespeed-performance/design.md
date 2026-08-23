## Context

`https://yewton.net/` は Hugo 0.159.1 + HugoBlox v0.11.0（`github.com/HugoBlox/kit/modules/blox`）+ `analytics@v0.3.1` を Hugo Modules 経由で取り込み、Netlify にデプロイされる静的サイト。`_headers` は `static/_headers` で手動管理。

Lighthouse Desktop（v13.4.1, 2026-08-23 計測）の結果は Performance 92・LCP 1.4s・CLS 0.097。主要な改善余地は:
1. 全アセット `Cache-Control: max-age=0, must-revalidate` で再訪問時キャッシュが効かない
2. Inter Variable woff2 325KB がインライン読み込みされ LCP を遅延させる
3. hero 背景画像 `home-bg.jpg` が `<link rel="preload">` されておらず LCP 経路が遅い

HugoBlox は font の `native` 切替と `layouts/_partials/hooks/head-end/` 経由の head 拡張ポイントを提供しており、これらと Netlify の `_headers` 機能の組み合わせで HugoBlox 標準機能の範囲内で改善できる。

## Goals / Non-Goals

**Goals:**
- HugoBlox 標準機能の範囲内で Performance 95+・LCP 1.2s 以下・CLS 0.05 以下を達成する
- ハッシュ付きアセットを 1 年キャッシュし再訪問時のロードを最小化する
- HugoBlox テンプレートの上書きを伴わず、将来の HugoBlox アップデートでコンフリクトしない

**Non-Goals:**
- HugoBlox 内部実装の変更（フォント読み込みの最適化、ヒーロー画像の LCP 制御等）
- Hugo Modules 配下のテンプレート上書き（`_vendor` 化されない仕様により上書き不可）
- Inter フォントのサブセット化（ネイティブ切替で十分なため別途検討）
- gtag.js の lazy load（既に `async` 付与済み、計測取りこぼしリスク回避）
- 記事ページの LCP 改善（home-bg.jpg は home 限定）

## Decisions

### D1: Cache-Control は Netlify `_headers` で設定
Hugo 本体にキャッシュヘッダー設定機能はない。Netlify の `_headers` 形式（既存ファイルへの追記）で対応する。

各パスの分類と判断根拠:
- `/css/_entry.*.css`、`/js/*.min.*.js`、`/assets/js/preact-built/*`、`/dist/font/*`: ファイル名に SHA-256 ハッシュが含まれる。`immutable` を併用して 1 年キャッシュ。
- `/media/*`、`/*.webp`、`/*.jpg`、`/*.png`: ハッシュなしの静的アセット。コンテンツ差し替え可能性を考慮し 30 日（`max-age=2592000`）。
- HTML（`/*`）: 常に最新を配信する必要があるため `max-age=0, must-revalidate` を維持。

代替案（不採用）: `Cache-Control: public, max-age=300, stale-while-revalidate=86400` を HTML に付与 → Netlify CDN は HTML をキャッシュしないため効果薄。

### D2: Inter フォントはネイティブ切替（`font: native`）で十分
`site.Params.hugoblox.typography.font = "native"` を設定すると HugoBlox の `site_head.html` は `@font-face` ブロックを出力しない（HugoBlox 標準機能、テンプレート上書き不要）。

Inter Variable woff2 は 325KB で woff2 形式で既に圧縮済みのため、サブセット化しても削減率は限定的（Lighthouse 計測上 Inter は 1 リクエスト 0ms 扱い）。ネイティブ切替でリクエスト自体を消す方が効果が大きい。

代替案（不採用）: Inter のラテン文字サブセット化 → HugoBlox が `assets/dist/font/Inter.var.woff2` を管理しており、サブセット化するには Hugo Modules の `overrides/` 機構または `assets/` 上書きが必要。ネイティブ切替で十分なため見送り。

### D3: LCP プリロードは HugoBlox の head-end フックで実装
HugoBlox の `site_head.html` は `{{ partial "functions/get_hook" (dict "hook" "head-end" "context" .) }}` で `layouts/_partials/hooks/head-end/` 配下のファイルを読み込む（`get_hook.html` で実装）。ここに `preload-lcp.html` を追加すれば標準的な拡張ポイントで head 末尾に挿入できる。

`{{ if .IsHome }}` でホームページに限定する。記事ページでは `/media/home-bg.jpg` のプリロードは意味がない（404 のアセットをプリロードさせるリスクもある）ため必須。

代替案（不採用）: `layouts/_partials/site_head.html` のローカル上書き → Hugo Modules は `_vendor` 化されないため、上書きしても HugoBlox アップデート時に上書きファイルが削除される可能性がある。フック機構の利用が安全。

## Risks / Trade-offs

- **R1: HTML キャッシュを `must-revalidate` のままにすると Lighthouse 再訪問スコアへの効果が見えない** → Netlify Edge は HTML をキャッシュしないため、ハッシュ付きアセットの長期化が実ユーザの再訪問効果の本質。Lighthouse は初回訪問のみ計測するため、再訪問効果は合成テスト（`curl` で検証）で担保する。
- **R2: ネイティブフォントへの切替でデザインが変わる** → HugoBlox は `--hb-font-size-base` 等のトークンでフォントに依存しないサイジングを行っている。デザイン崩れは CV ページ・記事ページを目視確認する。
- **R3: `home-bg.jpg` プリロードの失敗** → Hugo が誤ったパスを出力するリスク。`hugo --gc --minify` でビルドし `public/index.html` に `<link rel="preload" ...>` が出力されることを確認する。

## Migration Plan

- デプロイ: GitHub の main ブランチへの push で Netlify が自動ビルド & デプロイ。
- ロールバック: 直前の commit を revert（3 ファイル追加のみ）。
- 検証: Lighthouse を再実行し Performance 95+・LCP 1.2s 以下・CLS 0.05 以下を確認。`curl -I` で `Cache-Control` ヘッダーが期待値どおりであることを確認。
- 観測: 本番デプロイ後 24 時間、Netlify Analytics でキャッシュヒット率（`X-Cache: HIT` 比率）を確認。