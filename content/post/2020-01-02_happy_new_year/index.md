+++
title = "久々の更新"
author = ["yewton"]
date = 2020-01-02
lastmod = 2020-01-10T09:25:36+09:00
slug = "happy-new-year"
tags = ["emacs", "hugo", "ox-hugo"]
categories = ["雑記"]
draft = false
+++

最近仕事用のmacを新調して環境を作り直す機会があったことと、 2020 年が始まるということもあり、久々にサイトを更新することにした。

[オープン職務経歴書](/cv/)を書きたかった、というのもある。

更新にあたっていくつか困難があった。

**まず、このサイトのソースが最近の Hugo では動かせなくなっていた。**

このサイトのテーマには [Icarus](https://github.com/digitalcraftsman/hugo-icarus-theme) を使わせていただいているのだが、最終更新が 2017 年となっており、
Hugo 0.55.0 以降で動かなくなってしまっていたり、 Deprecated Warning が出るようになっていた。

[PRは出されている](https://github.com/digitalcraftsman/hugo-icarus-theme/pull/124) のだけれど、マージされる様子が無い。

そもそも、当時のバージョンに対してカスタマイズしたレイアウトを作ってしまったので、
単純なテーマの更新だけでは追随できない。

どのバージョンなら動くのか突き止めるのも骨が折れた。
[asdf-gohugo](https://bitbucket.org/mgladdish/asdf-gohugo) を入れて、少しずつバージョンを上げながら確認していった。

```sh
asdf plugin add hugo https://bitbucket.org/mgladdish/asdf-gohugo
```

**そして、そもそも Hugo の使い方を忘れていた。**

[Hugo 導入記事](/2016/02/02/blog-with-hugo/) や [Hugo 用 Emacs ライブラリ](/2016/01/26/hugo-el/) を書いておいてなんだが、当時から3年も経ってほとんど忘れてしまっていた。

そこで改めて調べてみると、イマドキは org-mode で書くことも出来るらしい。

この3年ですっかり org-mode 無しでは生きられない体になってしまっていたし、
せっかくなので本記事からは [ox-hugo](https://ox-hugo.scripter.co/) を使って書くことにする。

使い始めるにあたって色々調べたり考えたりすることも多かったので、そのうち記事にしようと思う。
結論としては、org-mode と Hugo の組み合わせは最高だし、その橋渡しをしてくれる [ox-hugo](https://ox-hugo.scripter.co/) は本当にグッジョブだということ。

...

そんなこんなで色々対応していたものの、デプロイする仕組みなどは当時と変わりなく動いたのでその点は助かった。

org-mode で書けるようになったことだし、少しは記事を書いていきたい所だが、まずは Hugo の最新版に追従する作業が待っている…。
