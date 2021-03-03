---
title: "Academic 4.8.0 から Wowchemy 5.0.0 へのアップデート記録"
author: ["yewton"]
date: 2021-02-28T00:37:00+09:00
mylastmod: 2021-03-03T21:56:22+09:00
slug: "academic4-to-wowchemy5"
categories: ["Webサイト運用"]
draft: false
---

<div class="ox-hugo-toc toc">
<div></div>

<div class="heading">&#30446;&#27425;</div>

- [背景](#背景)
- [アップデート手順](#アップデート手順)
- [(おまけ)Chromebook でローカルサーバーにアクセス出来ないとき](#おまけ--chromebook-でローカルサーバーにアクセス出来ないとき)
- [終わりに](#終わりに)

</div>
<!--endtoc-->


## 背景 {#背景}

2021年2月25日に Wowchemy (旧 Academic) の 5.0.0 正式版が[発表された](https://wowchemy.com/blog/v5.0.0/)。
4.8 が2020年3月だったので、実に1年ぶりのアップデートとなる。

Academic から Wowchemy にリブランディングされただけではなく、後方互換性の無い大幅な変更が行われている。


## アップデート手順 {#アップデート手順}

公式には [Beta 0](https://wowchemy.com/blog/v5.0.0-beta.0/)、[Beta 1](https://wowchemy.com/blog/v5.0.0-beta.1/)、[Beta 2](https://wowchemy.com/blog/v5.0.0-beta.2/)、[Beta 3](https://wowchemy.com/blog/v5.0.0-beta.3/) を順に辿るとよい、とあるものの、実際のところ先の変更が後で無意味になったりしたので、改めてまとめる。

まず、最大の変更点である [Hugo Modules](https://gohugo.io/hugo-modules/) への対応。

1.13 以上の go をインストールした後に、
Wowchemy が対応しているバージョンの Hugo を入れる。

これがパッと分からなかったのだけれど、
[Beta 3 のリリースノート](https://wowchemy.com/blog/v5.0.0-beta.3/) から Hugo 0.80.0 が最新動作確認版であることが分かり、また git 上で [5.0.0](https://github.com/wowchemy/wowchemy-hugo-modules/tree/v5.0.0) と [Beta 3](https://github.com/wowchemy/wowchemy-hugo-modules/tree/v5.0.0-beta.3) の指す先が同じであることから、
5.0.0 でも Hugo 0.80.0 が使えることが分かった。

例によって extended 版が必要なので、もし arm 版が欲しい場合は <https://github.com/gohugoio/hugo/tree/v0.80.0> から zip をダウンロードしたのちに自身でビルドする必要がある:

```sh
go build --tags extended
```

用意が出来たら、従来のテーマを削除する:

```sh
git submodule deinit themes/academic
git rm themes
```

そして、 `config/_default/config.toml` から `theme = "academic"` の記述を削除し、以下を最下部に追加する:

```toml
[module]
  [[module.imports]]
    path = "github.com/wowchemy/wowchemy-hugo-modules/wowchemy-cms"
  [[module.imports]]
    path = "github.com/wowchemy/wowchemy-hugo-modules/wowchemy"
```

その後に、モジュールを初期化する:

```sh
hugo mod init github.com/yewton/yewton.net
```

これだと開発版が入ってしまうため、特定バージョンを入れ直す。

Wowchemy のドキュメントにはコミットハッシュ指定で入れるように書いてあるので、何でバージョン指定出来ないんだろう、と思い調べてみたところ、どうやらバージョン指定で入れられるようなモジュール設定になっていないらしかった
(参考: [Goメモ-35 (モジュールのメジャーバージョンを２以降にした場合の取り扱い方について) - いろいろ備忘録日記](https://devlights.hatenablog.com/entry/2019/12/20/132730) )。

というわけで 5.0.0 のコミットを指定して入れる…と、ここで罠があって、 **コメント機能を使っている場合、以下の不具合の為にエラーになる** :

[fix: comment provider never found by arhohuttunen · Pull Request #2165 · wowchemy/wowchemy-hugo-modules](https://github.com/wowchemy/wowchemy-hugo-modules/pull/2165)

そのため、この修正が取り込まれたコミットを指定してやる:

```sh
hugo mod clean
hugo mod get github.com/wowchemy/wowchemy-hugo-modules/wowchemy@3cf9f6c
hugo mod get github.com/wowchemy/wowchemy-hugo-modules/wowchemy-cms@3cf9f6c
hugo mod tidy
```

画像置き場が `/static/img/` `/static/media/` に変わるので、合わせてリネーム+置換を行う。自分は [rg.el](https://github.com/dajva/rg.el) の wgrep でシュッとやってしまった。

以降、細かい非互換変更への対応。

`content/authors/admin/_index.md` の `name` を `title` に置換する。

`config/_default/params.toml` に以下を追加する:

```toml
# Content Management System
[cms]
 # Enable the admin panel. See https://wowchemy.com/docs/install/
 branch = "master"
 local_backend = false

# Icon Pack Extensions
[icon.pack]
 ai = true  # Enable the Academicons icon pack https://jpswalsh.github.io/academicons/
```

また、 `search`, `comments`, `map` の設定キー `engine` が `provider` に変わり、また設定値が数値から文字列に変わっている。これもそれぞれ変更が必要。

記事中での `alert` ショートコードが非推奨になるので、 `callout` に一括置換。

以上で移行は完了した。


## (おまけ)Chromebook でローカルサーバーにアクセス出来ないとき {#おまけ--chromebook-でローカルサーバーにアクセス出来ないとき}

[Chromebookでローカルサーバーを立てる | Chromebook情報ポータル「chromebooker」](https://chromebooker.net/topics/qzy312ksxc/) にある通り、
`localhost` でアクセス出来るときもあるのだけれど、出来なくなる場合もある
(たぶん初回だけアクセス出来る？)。

そんな時は昔ながらの <http://penguin.linux.test:1313/> でアクセスすれば良いのだけれど、デフォルトでは hugo server は `127.0.0.1` にバインドしているのでそのままでは拒否される。

以下のように全アクセスを許可してやれば良い(信頼出来るネットワークなら):

```sh
hugo server -DF --bind 0.0.0.0
```


## 終わりに {#終わりに}

終わってみれば、大した手間もかからずに移行出来た。

とはいえ、公式ドキュメントだけだと若干引っ掛かる部分もあったので、この記録が誰かの役に立てば幸い。
