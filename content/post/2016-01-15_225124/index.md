+++
banner = "banners/hina.jpg"
categories = ["雑記"]
date = "2016-01-15T23:00:00+09:00"
slug = "diary"
tags = ["blog", "hugo", "scrum", "javascript", "moment.js", "emacs", "web-mode", "go"]
title = "忙し過ぎてお弁当も食べられなかった日"

+++

最近チームが新体制に移行するという時期で色々バタバタしていて、
立ち上げに伴う開発フローの整備とか、JIRA(プロジェクト管理ツール)の設定とか、
ワーキングアグリーメントの検討とかで、頓に忙しい。

別に、自分の役割はスクラムマスターではなく、いちメンバーに過ぎないのだけれど、
認定スクラムマスターを取得した身としては色々手を焼かずにはいられず、
色んな雑務を買って出ているという状況。

それもこれも、スプリント中は〈全力疾走〉するのがスクラムである、
というスクラムのあるべき姿を目指して、自分が全力で開発に打ち込めるようにするため。

…と思ってたけど、最近は *チームのパフォーマンスを引き出す* っていうことに全力になっていて、
それはそれで楽しくなってきたので、スクラムマスターもいいかなー、なんて思い始めた。

## サイト改修 ##

今日もサイトいじりに勤しんだ。

いじるときはEmacsを使うんだけれど、自分は適当なのでHTML中にJavaScriptを埋め込んだりする。
するとhtml-modeだとシンタックハイライトとかが無くて辛いことになるので、
[web-mode](http://web-mode.org/)を使ってる。

HTMLファイルはCSSファイルは、始めから関連付けておくといいと思う。

### gravatar対応 ###
静的画像だけでなく、gravatarのアイコンを使えるようにした。

ついでに、画像を丸く表示するようにした。

[gravatarに対応している他のテーマ(greyshade)](https://github.com/cxfksword/greyshade/blob/28fb061bb674a2add89724dfbbf167f88f381d40/layouts/partials/header.html)と、
[How to Round Gravatar Images in WordPress](https://tdwp.us/round-gravatar-images-wordpress/)を参考にした。

### 記念日対応 ###

最初は[Hatena::Counting](https://counting.hatelabo.jp/)とか類似のサービスを利用しようと思ったんだけど、
あんまりシンプルなのが無かったので自作した。

自作といったって、[Moment.js](https://momentjs.com/)というとても便利なライブラリがあったので、
それを使って適当にJavaScriptを埋め込んだだけ。

[template - The Go Programming Language](https://golang.org/pkg/text/template/)とか
[Hugo - Go Template Primer](https://gohugo.io/templates/go-templates/)あたりも参考にした。
