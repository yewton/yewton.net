---
title: "Wowchemy 5.0.0 から 5.2.0 へのアップデート記録"
author: ["yewton"]
date: 2021-06-02
mylastmod: 2021-06-02T00:41:33+09:00
slug: "wowchemy-500-to-520"
categories: ["Webサイト運用"]
draft: false
---

## 背景 {#背景}

[2021年5月4日に 5.1.0](https://wowchemy.com/blog/v5.1.0/) が、[同年5月26日に 5.2.0](https://wowchemy.com/blog/v5.2.0/) が、それぞれ発表された。

[前回](/2021/02/28/academic4-to-wowchemy5/) で [Hugo Modules](https://gohugo.io/hugo-modules/) への対応が済んでいるので、サクッとアップデート出来るようになった。

多少、後方互換性の無い変更があるので、対応の記録を残す。


## アップデート手順 {#アップデート手順}

まず、 Hugo 自体を、対応最新バージョンである v0.83.1 に上げる
( [リリースノート](https://github.com/wowchemy/wowchemy-hugo-modules/releases/tag/v5.2.0) によると、 `Hugo Extended v0.81.0-v0.83.1` に対応している模様 )。

`netlify.toml` の変更も忘れずに。

その後、リリースノートに記載の通り以下を実行する:

```sh
hugo mod get github.com/wowchemy/wowchemy-hugo-modules/wowchemy@89d079b
hugo mod get github.com/wowchemy/wowchemy-hugo-modules/wowchemy-cms@89d079b
```

これだけだと、 `go.sum` にゴミが残るので、更に [hugo mod tidy](https://gohugo.io/commands/hugo%5Fmod%5Ftidy/) しておくとよい。

[Wowchemy v5.1.0 の Apply Breaking Changes](https://wowchemy.com/blog/v5.1.0/#apply-breaking-changes) に従い、以下を行う:

-   `assets/images/` を `assets/media/` にリネーム
-   デフォルトソーシャルシェア用画像の名前を `assets/media/sharing.*` に設定し、 `sharing_image` の設定を `config.yaml` から削除
-   `static/media/` 以下のメディアファイルを、 Hugo のメディア処理システムの対象になるように `assets/media/` に移動する

以上で、アップデートは完了となる。簡単!
