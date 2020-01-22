---
title: "GitHub Pages + Cloudflare から Netlify に移行した"
author: ["yewton"]
date: 2020-01-13
mylastmod: 2020-01-23T04:19:50+09:00
slug: "hugo-netlify"
tags: ["Hugo", "Netlify", "Cloudflare", "GitHub Pages"]
categories: ["Webサイト運用"]
draft: false
image:
  caption: Background image by <a href="https://pixabay.com/users/PIRO4D-2707530/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=1687319">PIRO4D</a> from <a href="https://pixabay.com/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=1687319">Pixabay</a>
---

[この記事にあるとおり](/2016/02/02/blog-with-hugo/) 、当初このサイトは [GitHub Pages](https://pages.github.com/) でホストされ、
[Cloudflare CDN](https://www.cloudflare.com/) を利用して配信していた。

だが先日 [テーマを変えた](/2020/01/06/made-with-academic/) ときに、既に使われていないリソースがそのまま残ってしまっていることに気がついた。
単純に `/public` 以下を `git add -a` しているだけなので、明示的に消さなければ反映されなくて当然だった。

デプロイスクリプトを見直してもよかったが、 [Hugo](https://gohugo.io/) 自体が [Netlify](https://www.netlify.com/) でホストされているし、
[Netlify でホストするときの公式ドキュメントも用意されている](https://gohugo.io/hosting-and-deployment/hosting-on-netlify/) し、
[アセットの最適化機能](https://docs.netlify.com/site-deploys/post-processing/#post-processing-features) とかもついてくるし、何より [無料だし](https://www.netlify.com/pricing/) 、
ということで、移行することを決めた。

<div class="ox-hugo-toc toc">
<div></div>

<div class="heading">&#30446;&#27425;</div>

- [デプロイ手順](#デプロイ手順)
- [ドメイン移行手順](#ドメイン移行手順)
- [GitHub Pages で提供していたその他のリポジトリのリダイレクト設定手順](#github-pages-で提供していたその他のリポジトリのリダイレクト設定手順)
- [その後](#その後)

</div>
<!--endtoc-->


## デプロイ手順 {#デプロイ手順}

[公式ドキュメント](https://gohugo.io/hosting-and-deployment/hosting-on-netlify/) を超ざっくり要約すると、

1.  [Netlify でアカウント作成](https://app.netlify.com/)
2.  [`netlify.toml`](https://gohugo.io/hosting-and-deployment/hosting-on-netlify/#configure-hugo-version-in-netlify) をドキュメントを参考に配置
3.  連携する GitHub リポジトリを選択
4.  `Deploy site` ボタンをポチる

**以上。**

これだけで `hoge.netlify.com` でアクセス出来るようになる。いい時代になった。


## ドメイン移行手順 {#ドメイン移行手順}

ただドメインも移行する場合はもう一手間必要で、[Netlify の公式ドキュメント](https://docs.netlify.com/domains-https/custom-domains/configure-external-dns/) に従って
DNS レコードの設定をする必要がある。

[Cloudflare](https://www.cloudflare.com/) の場合は、以下のように Cloudflare のダッシュボードで、
各ドメインが Netlify への `CNAME` となるように設定する。

{{< figure src="2020-01-13_15-37-38_cloudflare.png" >}}

このとき、 **`Proxy status` が `DNS only` となるように**
(雲のアイコンがオレンジではなくグレーになるように) 設定する必要がある。

さもないと、 Cloudflare の CDN 機能が間に挟まってしまい、 Netlify 側から認識されない。

なお、 **`DNS only` にした時点で Cloudflare が発行した SSL 証明書は無効になる** 。
そのため、ここからなるべく早く [Netlify 側の SSL 証明書の設定](https://docs.netlify.com/domains-https/https-ssl/#certificate-service-types) をした方がいい。

当サイトの場合は設定してから1時間もしない内に SSL 証明書が有効になっていた。


## GitHub Pages で提供していたその他のリポジトリのリダイレクト設定手順 {#github-pages-で提供していたその他のリポジトリのリダイレクト設定手順}

サイトだけなら以上で万事完了なのだけれど、 GitHub Pages を利用していた場合の注意点として、
`github.io` リポジトリで `CNAME` を設定していた場合、それ以外で Pages 機能を利用しているリポジトリも同様のドメインでアクセスされるようになっている。
つまり、 `github.io` を Netlify へ向けた時点で、 **それ以外のリポジトリが全てリンク切れとなる。**

そのため、適切にリダイレクトするようにしなければならない。

[Netlify の公式ドキュメント](https://docs.netlify.com/routing/redirects/) に従ってやればいい…と思ったのだけれど、
原因は不明だが **`netlify.toml` で設定しようとしても反映されなかった** 。

最終的な `_redirects` ファイルの内容は以下のようになった:

```text
https://yewton-net.netlify.com/* https://www.yewton.net/:splat 301!

/swagger-top-down-playground/* https://yewton.github.io/swagger-top-down-playground/:splat 301!
/dockerfiles/*                 https://yewton.github.io/dockerfiles/:splat 301!
/kafka-doc-ja/*                https://yewton.github.io/kafka-doc-ja/:splat 301!
```


## その後 {#その後}

移行ついでに [How to Configure Better Web Site Security with Cloudflare and Netlify | Okta Developer](https://developer.okta.com/blog/2019/04/11/site-security-cloudflare-netlify) を参考に
[Security Headers](https://securityheaders.com/) や [SSL Server Test](https://www.ssllabs.com/ssltest/index.html) で高評価になるように設定を見直したりなどした。

ただこれはあんまり理解しないで書かれてる通りやっただけなところがあるので、いつか改めて記事にしたい。

{{< figure src="2020-01-13_15-12-54_securityheaders.png" >}}

{{< figure src="2020-01-13_15-15-03_qualys.png" >}}
