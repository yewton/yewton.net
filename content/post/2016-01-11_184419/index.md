+++
date = "2016-02-02T18:44:21+09:00"
categories = ["技術系"]
tags = ["github", "hugo", "cloudflare", "ssl", "dns"]
title = "Hugo + GitHub Pages でお手軽にブログを始めよう"
slug = "blog-with-hugo"

mylastmod = 2020-01-13T15:43:21+09:00
+++

{{< lastmod >}}

{{< callout note >}}
2020年1月現在は、 [Netlify を使っています](/2020/01/13/hugo-netlify/) 。
{{< /callout >}}


## はじめに ##

この記事は「いい感じのブログを無料で手軽に作れないかなー」、と思っている人が主な対象です。

〈いい感じ〉というのが抽象的ですが、以下のような欲求をイメージしています:

- [Top Open-Source Static Site Generators - StaticGen](https://www.staticgen.com/) にあるような静的サイトジェネレータがいい
- サイト生成は速ければ速いほどいい
- モダンでレスポンシブルなのがいい
- 記事を書いたら即確認出来るのがいい
- GitHub Pagesに簡単にデプロイ出来るのがいい
- 独自ドメインは使いたい
- ブログにありがちなコメント機能とかアクセス解析とかが出来るといい
- HTTPSなのがいい
- HTTP/2なのがいい

## サイトの方式を考える ##

手軽に始めたいので、準備はなるべく少ない方が嬉しいですね。

WordPress みたいな動的な方式はサーバを用意しないといけないので手間ですし、場合によっては金もかかります。

一方の静的サイト配信であれば、必要なのは HTML を配信出来る場所だけです。
GitHub Pages や類似のサービスを使えば無料で利用できますね。

## 静的サイトジェネレータを選ぶ ##

というわけで静的サイトを作成するためのジェネレータを選びます。

[Top Open-Source Static Site Generators - StaticGen](https://www.staticgen.com/) の中から適当に選びましょう。

[Jekyll](https://jekyllrb.com/) や [Octopress](http://octopress.org/) はメジャーっぽいですが検索してみると、何だか遅いみたいです(使ったことない)。

[GitBook](https://www.gitbook.com/) はドキュメントを書くのには向いています(自分も利用しています)。ですがブログを書くためのものではありません。

[Hexo](https://hexo.io/) か [Hugo](https://gohugo.io/) あたりがよさそうです。
これら二つはコンセプトもよく似ているし、最早好みの世界ですね。自分は何となく Hugo を使っています。

## Hugo ##

![Hugoのロゴ](/media/2016-01-11_184419/hugo.png)

[Hugo](https://gohugo.io/) は超高速でシンプルかつ柔軟な静的サイトジェネレータだそうです。
Go言語で書かれていて、インストールはとってもカンタン。すぐに使えます。

使い方もとってもカンタン…というワケではないかもしれません。
といっても、それは Hugo が殊更難しいというワケではなく、
そもそも静的サイトジェネレーターというモノを理解している必要がある、ということだと思います。

ブログを書く、ということだけに目的を絞れば、覚える必要がある用語は
*Content*, *Themes*, *Taxonomies* の3つだけです。

### Content ###

ブログで言えば記事のことです。

Content がどのように表示されるかは使用しているテーマによりますが、
ブログ用テーマのほとんどは `post` ディレクトリ以下にあるファイルを記事として扱うようです。

### Themes ###

そのまま、サイトに適用するテーマです。

Hugoの場合、テーマはそのサイトの構成まで決めてしまいます。
独自にテンプレートを書くことでカスタマイズ出来るとはいえ、目的に沿ったテーマを選択するのが無難です。

テーマ選択の方法については後述します。

### Taxonomies ###

やけに難しい単語ですが、分類方法のことです。
ブログなら **カテゴリ** や **タグ** といった類のものです。

幸い、カテゴリとタグによる基本的な分類であればデフォルトで対応しているため、あまり意識する必要はありません。
カテゴリやタグのことを Taxonomies と呼ぶことだけ押さえておけばOKです。

## サイトを作る ##

早速サイトを作っていきましょう。

[Hugo - Hugo Quickstart Guide](https://gohugo.io/overview/quickstart/) に従えば基本的に迷うことは無いと思います。
ここでは、適当にサイトを作って、ひとつ記事を書いてみて、それがブラウザで確認出来るようになれば大丈夫です。

ただし、 **日本語に対応させる設定は追加で必要** です。

### 日本語対応 ###

ありがたいことに、Hugoは日本語や中国語の為の特別な設定を用意してくれています。
設定ファイルに `hasCJKLanguage = true` という行を追加しましょう。

これをやらないと、一覧用に記事を自動で切り詰めてくれる機能や、
読み終えるまでの予想時間の計算が滅茶苦茶になってしまいます。

## テーマを決める ##

次はサイトに適用するテーマを選択しましょう。

[Hugo Themes Site](https://themes.gohugo.io/) で実際の例を見ながらテーマを選ぶことが出来ます。
ブログ用のテーマのみに絞って見たい場合は [こちら](https://themes.gohugo.io/tags/blog) からどうぞ。

注意点として、Hugoには記事を読み終えるまでの時間やおおよその文字数を表示する機能が組込まれているのですが、テーマがサポートしていない場合は表示されません。
このような機能を使いたい場合は、テーマでサポートされているかを確認しましょう。

このサイトは [Icarus](https://themes.gohugo.io/hugo-icarus/) を使っています。
ショーケースの中では使える機能が一番多いと思うので、どういうことが出来るのか知るには丁度よいテーマだと思います。

## favicon を置こう ##

デフォルトでは Hugo の favicon が表示されてしまうので、カスタマイズしましょう。
[Favicon &amp; App Icon Generator](https://www.favicon-generator.org/) などを使うとよいです。

生成した `.ico` ファイルを `static` 直下に配置すればOKです。

## 記事を書く ##

記事はMarkdownで書きます。

ここで困るのは、Markdownにもいくつか方言があることです。
何を参考に書けばいいか迷ってしまいますね。

HugoではMarkdownの処理に [russross/blackfriday: Blackfriday: a markdown processor for Go](https://github.com/russross/blackfriday) を使っています。
Blackfriday特有の書式(脚注など)もある為、こちらを参照しながら書くとよいでしょう。

### 記事のファイル名について ###

日記や思い付きで即興で記事を書く場合、いちいち被らないようにファイル名を考えるのは面倒ですよね。

Hugoの場合ファイル名は管理上の問題でしかないので、適当に日付やタイムスタンプでも入れておけばよいです。
デフォルトではファイル名がURLに使われますが、記事毎の設定で `slug = "hogehoge"` のように設定しておけば、
実際のURLは `/post/hogehoge` のようになります。

また、以下のように設定ファイルに書いておけば、年月日がURLのプレフィクスに付くので被る心配もありません:

```
[permalinks]
    post = "/:year/:month/:day/:slug"
```

## コメント対応 ##

なんとなくコメント欄があるとオープンな感じでいいですよね？
ということでコメントにも対応させてみましょう。

Hugo自身が[Disqusに対応している](https://github.com/spf13/hugo/blob/cd36d752a3e8e2b75965fe281e6466d7a274cd94/tpl/template_embedded.go#L131-L145)ので、
[Disqus](https://disqus.com/)を使いましょう。
ただし、Disqusによるコメント対応も、テーマによってはサポートされていない場合があるので要注意です。

何よりもまず Disqus への登録です。
[ヘルプ](https://help.disqus.com/customer/portal/articles/466182-publisher-quick-start-guide)を見ながら行いましょう。
設定は特に必要ありませんので登録だけ済ませれば一旦OKです。後から言語設定を日本語にしたりすることも出来ます。

登録が完了したら、登録時に入力した shortname (unique Disqus URL) を以下のように設定に追記しましょう:

```
disqusShortname = "sitename"
```

これだけでHugoの設定は完了です。(テーマが対応していれば)個別の記事ページにコメント欄が出現します。


## デプロイ ##

生成したブログは GitHub Pages でホスティングします。

`gh-pages` ではなく、ユーザや Organization のページとして公開する場合は、
[Hosting Personal/Organization Pages](https://gohugo.io/tutorials/github-pages-blog/#hosting-personal-organization-pages)に
何も考えずに従えばOKです。
例示されているスクリプトもそのままコピペで使えます。

実際にこのブログで使われているモノは全て[こちら](https://github.com/yewton/yewton-hugo)にあるので参考にしてください。

## 独自ドメイン対応 ##

github.com ドメインでホスティングするだけでよければ、ここから先の作業は不要です。おめでとうございます。

以降はHugoとは関係ない話が続きます。

既に独自ドメインを持っていて、それをブログでも使いたい場合は、
[CNAME](https://help.github.com/articles/adding-a-cname-file-to-your-repository/)の設定をGitHub上で行う必要があります。
[こんなファイル](https://github.com/yewton/yewton.github.io/blob/master/CNAME)を作ってあげて、
DNSレジストラ側で `CNAME` とか `ANAME` の設定を行ないます。

### サブドメイン vs ルートドメイン ###

`CNAME` の設定をするか `ANAME` の設定をするかは、サブドメインを使うかルートドメインを使うかの違いです。
GitHubのヘルプによると、[サブドメインを強く推奨している](https://help.github.com/articles/about-custom-domains-for-github-pages-sites/#subdomains)そうです。

サブドメインを推奨する理由は以下のようです:

- GitHubのCDNの恩恵を受けられる
- GitHub自体のIPアドレス変更に影響を受けない
- DoS対策がより効率的になるのでページロードが高速になる

というわけで、特別な事情がない限りはサブドメインを利用するのがよいでしょう[^1]。

[^1]: こう書いている自分も、最初は特に気にせずにルートドメインで登録してしまっていました。この記事を書きながら気づいて、慌てて `www` サブドメインに移行しました。

## HTTPS化 ##

ここまででブログとしての体裁は整いました。
ですが、折角つくったサイトですからHTTPSで配信したいですよね？しましょう。

これには[CloudFlare](https://www.cloudflare.com/)を利用出来ます。
[GitHub Pagesに設定しているカスタムドメインをHTTPS対応させる - 1000ch.net](https://1000ch.net/posts/2015/github-pages-custom-domain-in-https.html)が非常に参考になります。

設定が反映されるまでは最長1日程度かかりますので、ゆっくりと待ちましょう。

なおCloudFlareを利用すると、ついでに HTTP/2 も有効になります。
他にもJSの軽量化などの各種高速化の設定を利用出来ますので、設定項目を見てみるのもよいでしょう。

### 注意 ###

注意点として、この際に Flexible SSL を用いる場合は、あくまでユーザとCloudFlare間の通信が暗号化されるだけで、 **CloudFlareとGitHub Pages間の通信は暗号化されていません** 。
後者の間の通信は改竄されうる状態で、かつ、ユーザはそれを知る術が無いという状態に…。

[GitHub Pages Now (Sorta) Supports HTTPS, So Use It](https://konklone.com/post/github-pages-now-supports-https-so-use-it)に詳しく書かれていますが、
一応そういう状態であるということは認識しておきましょう。

## 終わりに ##

いい感じのブログを無料で手軽に作る方法についてまとめました。
改めてまとめてみるとそれなりにボリュームがあってちょっと大変でした…。

これからブログを始めるぞーと思っている誰かの役に立てばいいなーと思います。
