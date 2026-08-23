## Why

`https://yewton.net/` の Lighthouse Desktop 計測で Performance 92・LCP 1.4s・CLS 0.097 と境界線の結果であり、再訪問時のキャッシュが効かないため実ユーザの体感は計測値より悪い。HugoBlox 標準機能の範囲内で改善できる余地が大きいにもかかわらず、着手されていない。

## What Changes

- `static/_headers` に Cache-Control を追加（Netlify 標準機能）。HTML は `max-age=0, must-revalidate` のまま、ハッシュ付きアセットは `max-age=31536000, immutable`、画像は `max-age=2592000`。
- `config/_default/params.yaml` に `hugoblox.typography.font: native` を追加（HugoBlox 標準機能）。Inter Variable woff2（325KB）の読み込みを抑止し、LCP を短縮する。
- 新規フック `layouts/_partials/hooks/head-end/preload-lcp.html` を追加し、ホームページのみ `<link rel="preload" as="image" href="/media/home-bg.jpg" fetchpriority="high">` を head に挿入（HugoBlox `get_hook` 機構）。LCP 候補画像を早期取得させ LCP を短縮する。

## Capabilities

### New Capabilities
- `pagespeed-performance`: ホームページの Lighthouse Performance スコア・LCP・CLS の目標値、ハッシュ付きアセットの長期キャッシュ、ホームページ限定 LCP プリロード、Inter フォント非読み込みを要件として規定する。

### Modified Capabilities
- なし（既存の `blog` / `content-workflow` 仕様への影響なし）。

## Impact

- 影響ファイル: `static/_headers`（追記）、`config/_default/params.yaml`（1 行追記）、`layouts/_partials/hooks/head-end/preload-lcp.html`（新規）。
- HugoBlox テンプレートの上書きを伴わないため、HugoBlox モジュール更新でコンフリクトしない。
- 可逆性: すべて単一の差分（または単一ファイルの削除）で取り消せる。
- スコープ外: Inter フォントのサブセット化、`hb-head.min.js` の defer 化（Hugo Modules 上書きが必要）、gtag.js 遅延読み込み（既に `async` 付与済み）。