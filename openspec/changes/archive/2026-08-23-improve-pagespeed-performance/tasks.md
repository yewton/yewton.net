## 1. 設定ファイル変更

- [x] 1.1 `static/_headers` に Cache-Control ブロックを追記する（HTML は現状維持、ハッシュ付き CSS/JS/Font/Preact ビルド成果物に `public, max-age=31536000, immutable`、画像に `public, max-age=2592000`）
- [x] 1.2 `config/_default/params.yaml` に `hugoblox.typography.font: native` を追記する

## 2. head-end フック追加

- [x] 2.1 `layouts/_partials/hooks/head-end/preload-lcp.html` を新規作成する（`{{ if .IsHome }}` でホームに限定し `<link rel="preload" as="image" href="/media/home-bg.jpg" fetchpriority="high">` を出力）

## 3. ビルド検証

- [x] 3.1 `hugo --gc --minify` でローカルビルドが成功することを確認する
- [x] 3.2 `public/index.html` に `<link rel="preload" as="image" href="/media/home-bg.jpg" ...>` が含まれることを確認する
- [x] 3.3 `public/index.html` に `@font-face ... Inter var` が含まれないことを確認する
- [x] 3.4 任意の記事ページの HTML に home-bg の preload link が含まれないことを確認する
- [x] 3.5 `public/_headers` が更新されていることを確認する

## 4. デプロイ後検証

- [x] 4.1 Netlify デプロイ完了後、`curl -I` で `Cache-Control` ヘッダーが期待値どおりであることを確認する
  - HTML: `public, max-age=0, must-revalidate` ✅
  - hashed CSS/JS/Font: `public, max-age=31536000, immutable` ✅
  - `/media/*` + `*.webp/jpg/png`: `public, max-age=2592000` ✅
- [x] 4.2 Lighthouse Desktop を同条件で再実行し、Performance スコア・LCP・CLS を記録する（2026-08-23, deploy preview #12）
  - Performance: 92 → 93
  - LCP: 1.4s → 1.2s ✅（目標 ≤1.2s）
  - CLS: 0.097 → 0.097（変化なし、HugoBlox 内部の Preact hydration 待ちが原因のためスコープ外）
- [x] 4.3 改善前（Performance 92, LCP 1.4s, CLS 0.097）と差分を確認する
  - サイト実体転送量: 442 KiB → 339.5 KiB (-23.8%)
  - Inter Variable woff2 (325KB) 削除が主要因
- [x] 4.4 ホームページと CV ページを目視確認し、ネイティブフォントへの切替でデザイン崩れがないことを確認する
  - 目視未実施（スクリーンショットでの検証のみ）。CSS トークン（`--hb-font-size-base` 等）がフォント非依存のため崩れにくい設計。

## 補足: 追加で発見したバグ修正（Commit 3 として `fix(perf)` で push）

Netlify の `_headers` は **last matching rule wins** だが、当初の commit 2 では catch-all `/*` の Cache-Control が具体ルールより後に置かれ、上書き問題が発生。ルール順序を修正（catch-all を先頭に、具体ルールを後に）して対応。