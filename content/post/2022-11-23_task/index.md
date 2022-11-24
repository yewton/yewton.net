---
title: "タスクランナー Task の紹介スライドを書いた"
author: ["yewton"]
date: 2022-11-24T22:04:00+09:00
mylastmod: 2022-11-24T22:04:55+09:00
slug: "task"
tags: ["task", "taskfile", "make"]
categories: ["技術系"]
draft: false
url_slides: https://yewton.github.io/my-marp-slides/2022-11_task/PITCHME.html
---

[Spring Microservices in Action, Second Edition](https://amzn.to/3TSpkqM) の独習用に、 <https://github.com/yewton/my-smia> というリポジトリで、実際に AWS に Terraform で環境構築してデプロイするところまでをやっている。

その際に利用したタスクランナー [Task](https://taskfile.dev/) が、丁度いい感じに使えて良かった為、紹介するためのスライドを作った:

{{< icon name="external-link-alt" pack="fas" >}} [タスクランナー Task の紹介](https://yewton.github.io/my-marp-slides/2022-11%5Ftask/PITCHME.html)

スライドにも書いたが、以下の点が特に気に入っている:

1.  環境変数の設定・切り替えが簡単
2.  [Go Template](https://pkg.go.dev/text/template) が使える
3.  タスクの構造化が簡単・直感的
4.  無駄な実行を防ぐのが簡単
5.  [JSON スキーマが公開されており](https://github.com/SchemaStore/schemastore/blob/master/src/schemas/json/taskfile.json) 入力支援が効く

その他のツールとして YAML テンプレーティングツール [ytt](https://carvel.dev/ytt/docs/latest/) も便利だった。
こちらについてもその内紹介したい。
