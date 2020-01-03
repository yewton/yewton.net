+++
banner = "banners/ssl.jpg"
categories = ["雑記"]
date = "2016-01-18T00:06:20+09:00"
slug = "diary"
tags = ["ssh", "cloudflare", "github", "https", "cdn"]
title = "HTTPSに対応してついでにHTTP/2対応された日"

+++

## HTTPS + HTTP/2 対応 ##

世の中的にHTTPS対応してないといかんような気がしたので、
このサイトもHTTPSで配信するように対応した。

[GitHub Pagesに設定しているカスタムドメインをHTTPS対応させる - 1000ch.net](https://1000ch.net/posts/2015/github-pages-custom-domain-in-https.html)を大いに参考にさせていただいた。
もう本当に書いてある通りにすればいい。 [CloudFlare](https://www.cloudflare.com/) 様々である。

敢えて付け加えるとしたら、(CloudFlareのサイト上に注記されてはいるけども)Flexible SSLが実際に動くようになるまでは結構時間がかかること。
自分の場合は、都合7〜8時間程度かかった。
この間、ステータスは `AUTHORIZING CERTIFICATES` から `ISSUING CERTIFICATES` になり、最終的に `ACTIVE CERTIFICATE` になった。

この方法でサイトをHTTPS対応するのは簡単だけども懸念もあるらしい[^1]ので、
また今度この辺が実際どういう仕組みで動いてるのか調べてまとめたいと思う。

なお、CloudFlareを利用することで副次的に HTTP/2 にも対応することになった。
Chromeで [HTTP/2 and SPDY indicator](https://chrome.google.com/webstore/detail/http2-and-spdy-indicator/mpbpobfflnpcgagjijhmgnchggcjblin)
を入れてアクセスしたりすると分かる。

[^1]: [GitHub Pages Now (Sorta) Supports HTTPS, So Use It](https://konklone.com/post/github-pages-now-supports-https-so-use-it)

## DDD ##

開発チームビルディングの一環で、ドメイン駆動設計について改めて色々調べたりしている。

図やドキュメントに本質はないとはいえ、ユビキタス言語のサポートとして用語集を作ったり、
深いモデルの洞察のために図が役に立ったりするので、そのへんのサポートツールが欲しくなる。

色々探してたら[エヴァンス先生がマイクロサービスとの関わりについて講習してる動画](https://skillsmatter.com/skillscasts/6259-ddd-and-microservices-at-last-some-bounderies)があった。
境界づけられたコンテキストはサービスじゃないから、別に実サービスと一対一対応する必要はない。
例えば、複数のコンテキストにまたがってやりとりされるようなあるコンテキストのメッセージがあるなら、
それは Interchange Context (日本語で言うと相互連結コンテキスト？)のように別のコンテキストとして定義すればいいよね、みたいな話があった。

せっかく内製開発してて、スクラムやってて、物理的にステークホルダー達とも近い距離にいるのだから、
その恩恵を最大限に活かせるようにしたい。
そのためにDDDって実践的に役立つと思うので、もっと勉強しなければ。

勉強の一環として今度[DDD Alliance! ドメイン駆動設計のためのオブジェクト指向入門](http://ddd-alliance.connpass.com/event/25209/)に行ってみる。
日本でDDDのコミュニティってここくらい？な気がするので、勉強会の内容というよりはコミュニティの様子が気になるので。
