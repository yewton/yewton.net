+++

categories = ["Emacs"]
date = "2016-01-26T23:25:40+09:00"
slug = "hugo-el"
tags = ["hugo.el", "Emacs", "hugo", "EmacsLisp"]
title = "hugo.elを書いた"

+++

## #とは ##

[hugo.el](https://github.com/yewton/hugo.el)は、Hugoでサイト作成する際に便利な関数群を定義したパッケージ。
[marmalade-repo](https://marmalade-repo.org/)で公開されているので、パッケージの設定をすれば簡単にインストール出来るハズ。

## 作った動機 ##

何か〈物を書く〉という作業をする時、自分はEmacsを使っていて、可能な限りEmacsの中から出たくない。
HugoはCLIのインタフェースなので、ターミナルとEmacsを行き来することになってしまうのは辛い。

あと色んなテーマを試したい時に、いちいちテーマ名をコピペする必要があり、辛かった。
一覧の中からhelmで選択したかった。

…というように、Emacsの中で色々作業が完結するようにしたかった。

## 作ってみて ##

正直テーマのインストール補助機能はHugoを初めて触って、色々試してみたいフェーズでしか使わないので、
ほとんどの場合大した価値じゃないかもしれない。

個人的には、 `hugo-new-content` と、 `hugo-start-server`, `hugo-open-browser` が中々便利だなーと思って使っている。
適当に記事を書き始められるし、プレビューするためにターミナルで確認したアドレスをブラウザに打ち込むとかやらなくてよいので。

書き終えたあとに `hugo-deploy` でひょいっと公開出来るのも地味に便利。
ただ、デプロイスクリプトを呼び出してるだけなんだけどね…。

普段Emacsで物書きをするひとで、Hugoでブログ書こうと思う人は是非便利に使ってもらいたい。
そんな人あんまりいなさそうだけど…。

## TODO ##

- `hugo-open-browser` で今開いているcontentを直接開けたらより便利かもしれない
- `hugo-deploy` が同期処理になってて固まるので、非同期にする
- テスト全然書いてないので、テストを書いてバッジをつけて喜ぶ
