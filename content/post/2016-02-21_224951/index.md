+++

categories = ["技術系"]
date = "2016-02-21T22:49:52+09:00"
slug = "wordpress-http2-hhvm"
tags = ["wordpress", "https", "hhvm", "http/2", "nginx", "docker"]
title = "WordPressをHTTP/2+HHVMで動かす環境をdocker-composeで構築してみた"

+++

## TL;DR ##

[このリポジトリ](https://github.com/yewton/wordpress-nginx-http2-hhvm)を `clone` して `docker-compose up -d` して下さい
(要 [バージョン 1.6.0 以上](https://github.com/docker/compose/releases/tag/1.6.0))。
その後 `open "https://$(docker-machine ip default)"` すると、ブラウザでWordPressが立ち上がります。

![デモ](/media/2016-02-21_224951/wordpress.png)

## 動機 ##

WordPressをHHVMで動かしたら超速くなった!やったぜ!!という記事はをよく見るのだけれど、
具体的にどうやって構築しているのか解説している記事があんまり無かったので、実際にやってみることにしました。

ついでに、HTTP/2で提供出来るとイケてる気がしたので、併せて対応してみることにします。

## HHVM ##

![HHVM](/media/2016-02-21_224951/hugo.png)

[HHVM](https://hhvm.com/)は、Facebookがオープンソースとして開発している仮想実行環境で、
[PHP](https://php.net/)と[Hack](https://hacklang.org/)を動かすための環境らしいです。

まぁ、個人的にはHackもHHVMも **すごいPHP** ぐらいの認識しかありませんが…。
Hackは言語的にすごくて、HHVMは実行環境がすごい。今回用があるのはHHVMだけです。

## HTTP/2 ##

[HTTP/2](https://http2.github.io/)は… **すごいHTTP** です。

HTTP/1.xとの互換性を保ちつつ、効率化したもののようです。ヘッダの圧縮とか、リクエストの多重化とか。
[RFC7540 日本語訳](https://summerwind.jp/docs/rfc7540/)や[日本語のFAQ](https://http2.info/faq.html#who-made-http2)もあります。
自分は全然見てないけれど…必要になったら読みます。

とにかく、このプロトコルで配信するだけでより効率的でより早くなる、ということです。

### HTTP/2 の実装 ###

[nginx](https://nginx.org/en/)の1.9.5からHTTP/2をサポートしているようです[^1]。
[公式Dockerリポジトリ](https://hub.docker.com/_/nginx/)で配信されている最新イメージでもちゃんとサポートされていました。

他にも[Apache HTTP Server 2.4.17+](https://httpd.apache.org/)や
[DeNA](https://dena.com/intl/)の[H2O](https://h2o.examp1e.net/)など[色々ある](https://github.com/http2/http2-spec/wiki/Implementations)ようです。

今回はApacheよりは速かろうというのと、WordPress稼動の実例も多いことから、nginxを選択しました。適当。

[^1]: [HTTP/2 Supported with NGINX Open Source 1.9.5 | NGINX](https://www.nginx.com/blog/nginx-1-9-5/)

## 構成 ##

リライトルールとか複雑なリクエスト制御が不要なら、
HHVMに組込みのWebサーバーがあるのでそれを使うのが簡単だし、速度面でも問題なさそうです。

ただ、WordPressでパーマリンクを利用する場合はURLのリライトが必須です。

HHVMでもバーチャルホスト切ってリライトの設定するとか出来るみたいですが、
iniファイルに設定書いていくのは何だかしんどそうです。

他にもSSLとか静的ファイルの配信とか諸々考えると、餅は餅屋ということでリバースプロキシを立てた方がよさそうですね。

というわけで、リバースプロキシとして nginx を立てて、HHVMをFastCGIモードで起動してバックエンドとします。

今回はとにかくお手軽に手元で動かしてみたかったので、諸々Dockerで動かすことにしました。
雑に図解すると以下のような感じです:

![構成図](/media/2016-02-21_224951/structure.png)

FrontとかBackはDockerの[ユーザー定義ネットワーク](https://docs.docker.com/engine/userguide/networking/dockernetworks/#user-defined-networks)です。
[Docker 1.10.0](https://github.com/docker/docker/blob/master/CHANGELOG.md#1100-2016-02-04)で link に代わるものとして導入されたような気がします。
`/etc/hosts` じゃなくてDNSで名前解決出来るようになってて最高にハッピーですね。

FastCGIはUnixソケットで通信した方が速いと思うんですが、
今回は nginx と HHVM を別々のコンテナで動かすので、TCPで通信するようにしました。
同一システム上にFastCGIサーバとプロセスが稼動するって、実環境でもあんまり無いような気がするけど、どうなんだろう？

コンテナひとつひとつ立てていくのは辛いので、常套手段の `docker-compose` を使って作ります。

## 動かす ##

[このリポジトリ](https://github.com/yewton/wordpress-nginx-http2-hhvm)に実際に稼動するものが置いてあります。
`clone` して `docker-compose up -d` すれば、HTTP/2でWordPressが動いている様子が確認出来ます。

nginxやHHVMの設定はほぼデフォルトのままいじってないので、本気で動かすなら細かいチューニングは必要です。
が、全体の構成は実環境でもこのようになると思います。

![デモ](/media/2016-02-21_224951/wordpress.png)

## 終わりに ##

以上で、お手軽に手元でHTTP/2+HHVMなWordPressを試すことが出来るようになりました。

あとは実環境へのデプロイだけなんですが、さてどうしたものか。
`docker-compose.yml` からいい感じにデプロイしてくれるような何か、ありませんかねぇ。
