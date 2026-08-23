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

- [ ] 4.1 Netlify デプロイ完了後、`curl -I` で `Cache-Control` ヘッダーが期待値どおりであることを確認する
- [ ] 4.2 Lighthouse Desktop を同条件で再実行し、Performance スコア・LCP・CLS を記録する
- [ ] 4.3 改善前（Performance 92, LCP 1.4s, CLS 0.097）と差分を確認する
- [ ] 4.4 ホームページと CV ページを目視確認し、ネイティブフォントへの切替でデザイン崩れがないことを確認する