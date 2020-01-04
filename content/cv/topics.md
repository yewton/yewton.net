+++
weight = 2007
# Accomplishments widget.
widget = "blank"  # See https://sourcethemes.com/academic/docs/page-builder/
headless = true  # This file represents a page section.
active = true  # Activate this widget? true/false

title = "トピック"
subtitle = ""

[design]
  # Choose how many columns the section has. Valid values: 1 or 2.
  columns = "1"
+++

## 動画サムネイルシークの開発 {#動画サムネイルシークの開発}

各時点のサムネイルを選んでシークするという機能を、バックエンドからフロントエンドまで全て一人で対応した。

FFmpeg のシーンチェンジ検出結果を保存、配信し、それをネイティブクライアントが利用してシークする、というもの。

ネイティブ開発は初めてだったが、3ヶ月程で Android, iOS ともに対応することが出来た。

**ひとつの機能をバックエンドからフロントエンドまで一気通貫で全て独力で実装する** という貴重な経験だった。


## メディアコンテンツ投稿・変換・配信システムのエンハンス開発 {#メディアコンテンツ投稿-変換-配信システムのエンハンス開発}

S3 にファイルを直接アップロードすることによる負荷軽減や、 [imgix](https://www.imgix.com/) と連携した画像配信、
複数ファイルの非同期並列アップロード対応などのエンハンスを行った。

素の JS のままでは並列的な処理を書くのがしんどかったため、 React でリライトした。

それまで単一ファイルのみ受け付けていたものを複数対応させるには Rails モデルのリアーキテクトも必要だった。
無停止で機能をリリースするため、事前に用意した新たなデータ構造に徐々に移行するという段取りを立て、実行を主導した。


## Keen IO -> BigQuery 連携 {#keen-io-bigquery-連携}

[Keen IO](https://keen.io/) がそれまで提供していた BigQuery 連携機能を停止してしまったため(ベータとして提供されていたので仕方ない)、急遽対応したもの。

Keen が S3 にローデータを配置するのをトリガーとして Lambda を起動、 BigQuery にデータを連携する、というもの。 [node-lambda](https://www.npmjs.com/package/node-lambda) を用いて実装した。

突発対応で急拵えだったが特に問題なく稼動し、ことなきを得た。

自分の中の技術の引き出しを増やすいい経験だった。


## Grape + OpenAPI による高速開発推進 {#grape-plus-openapi-による高速開発推進}

3ヶ月で開発を完了させなければならず、極力無駄を省くために、 [Grape::Entity](https://github.com/ruby-grape/grape-entity) と [grape-swagger](https://github.com/ruby-grape/grape-swagger) を採用し、実装と同時に API 定義を生成し、
クライアント側はその定義を元にライブラリを自動生成して実装を進める、というアプローチで開発を進めた。

結果として上手くいき、無事に期間内にサービスインすることが出来た。

Open API を活用する好事例だった。


## 完全なプログラミング勉強会 {#完全なプログラミング勉強会}

シニア的な立場になってきたころ、全員の「よいコード」の認識を揃えるべく、 [CODE COMPLETE](https://amzn.to/35lI48z) を題材にした勉強会を行なった。
そのときの資料は今でも指針として利用されている。

持ちネタとして活用出来ている。

_(今は社内限な感じになってしまっているので、無難な内容にしてそのうち一般公開したい)_
