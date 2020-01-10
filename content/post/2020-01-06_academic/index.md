+++
title = "#MadeWithAcademic"
author = ["yewton"]
date = 2020-01-06
aliases = ["/2019/01/made-with-academic/"]
lastmod = 2020-01-10T14:43:23+09:00
slug = "made-with-academic"
tags = ["academic", "ox-hugo"]
categories = ["雑記"]
draft = false
+++

[Icarus](https://github.com/digitalcraftsman/hugo-icarus-theme) がメンテされておらず Hugo 0.55.0 以降で正しく動かなくなっていたので、
自力で Icarus にパッチを充てるか、別のテーマに乗り換えるかという選択を迫られていた。

せっかくなのでランディングページとブログを別にしたかったので、そういう柔軟性を備えている
[Academic](https://sourcethemes.com/academic/) というテーマを使うことにした。

テーマの変更にあたって様々な知見が得られたので、そのうち記事にしたい。

以下その候補:

-   [Page Bundles](https://gohugo.io/content-management/page-bundles/) への移行
-   [ox-hugo](https://ox-hugo.scripter.co/doc/why-ox-hugo/) と Academic が如何に相性がよいか
    -   auto weight あたりが最高
    -   `lastmod` の自動更新や [Front-matter Extra](https://ox-hugo.scripter.co/doc/custom-front-matter/#front-matter-extra) は Academic じゃなても最高の体験
-   カスタムウィジェットの作り方
-   ダークテーマ対応のシンタックスハイライトのやりかた
-   conf-toml-mode を [Front-matter Extra](https://ox-hugo.scripter.co/doc/custom-front-matter/#front-matter-extra) で使う方法
