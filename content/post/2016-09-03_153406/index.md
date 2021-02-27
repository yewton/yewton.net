+++
banner = "banners/atlassian-dragon-slayer-t-shirt.jpg"
categories = ["技術系"]
date = "2016-09-03T15:34:07+09:00"
slug = "how-to-get-atlassian-dragon-slayer-t-shirt"
tags = ["Atlassian"]
title = "Atlassian から無料でTシャツをもらう方法、あるいはドラゴンの倒し方"

+++

仕事で Atlassian 製品を使っていて、個人的にも便利なタスク・ドキュメント管理に欲しいな、と思い、
[Atlassian のスターターライセンスについてのドキュメント](https://ja.atlassian.com/licensing/starter#aboutthestarterprogram-8)を読んでいた。

すると気になるフレーズが目に入った。 **「ドラゴンズレアとは何ですか？」** ──

> 6 つのスターター ライセンスの統合スイートをセットアップすると素晴らしい結果になりますが、セットアップ手順は複雑で時間がかかります。アトラシアンでは、アトラシアン アプリケーション スイートを統合するため方法をドラゴンズレアという説明書にまとめました。また、この困難だけれども素晴らしい旅を完了された方全員向けに、限定版アトラシアン ドラゴン スレイヤー T シャツを提供しています。今すぐ冒険を始めましょう! 勇気がある方はドラゴンに立ち向かいましょう!

せっかくもらえるもんなら、とドラゴン退治をすることにした。

{{% callout note %}}
**2017/1/20 時点の注意**

どうやら現在、 **Tシャツを要求するためのページが 404 になってしまっている** ようす
(
[Dragon slayed, but can't get t\-shirt? \- Atlassian Answers](https://answers.atlassian.com/questions/43114478/dragon-slayed-but-cant-get-t-shirt) )
。

せっかくやってもTシャツはもらえないかもしれないので注意。
{{% /callout %}}

{{% callout note %}}
**2017/2/5 時点の情報**

どうやらサイトが復旧したようす
[You Slayed the Dragon \| Atlassian](https://www.atlassian.com/you-slayed-the-dragon)
{{% /callout %}}

## 環境要件 ##

ドキュメントには「最低 3 GB の RAM と 500 MB のアプリケーションファイル用の空き容量」とある。

しかし、実際にやってみたところ 3 GB では全てのアプリケーションを立ち上げることが出来なかった。
最終的に 6 つのアプリケーションを稼動させることになるのだが、 5 つが限界だった。
**最低 RAM は 4 GB は必要** だと思う。

またディスク容量に関しては、まっさらな環境に諸々構築したあとの総使用量が 6 GB 程だった。
こちらも **最低ストレージは 10 GB は必要** だと思う。

自分は [Google Cloud Platform](https://cloud.google.com/) を利用して構築した。
Google Compute Engine の `vCPU x 1` (メモリ 3.75 GB)の標準インスタンスを使ったが、
前述の通りメモリが足りなくなってしまったので、 `vCPU x 2` (メモリ 7.5 GB)のインスタンスを使う方がよいと思う。

OS は **Debian GNU/Linux 8 (jessie)** を使用した。

## Here Be Dragons ##

それでは [ドキュメント](https://confluence.atlassian.com/display/ATLAS/Here+Be+Dragons) に従って冒険を始めよう。

基本的には書かれてある通りにやればいいだけなのだが、いくつか自分が躓いたポイントがあるので紹介していきたい。

### [Dragons Stage 1 - Install JIRA](https://confluence.atlassian.com/display/ATLAS/Dragons+Stage+1+-+Install+JIRA) ###

#### Step 1. Install Java ####

まず Java を用意しなければならないのだが、なんと **Oracle JDK 1.7.x** でなければならない。
[1.7 系はすでにサポートが終了している](http://www.oracle.com/technetwork/jp/java/eol-135779-ja.html)のだが、
このクエストではこのバージョンしか想定していないようなので、大人しく従っておく。

``` bash
wget --header "Cookie: oraclelicense=accept-securebackup-cookie" http://download.oracle.com/otn-pub/java/jdk/7u79-b15/jdk-7u79-linux-x64.tar.gz
```

Debian で Oracle Java を利用するには [JavaPackage - Debian Wiki](https://wiki.debian.org/JavaPackage "JavaPackage -
Debian Wiki") にあるような作業が必要になる。

``` bash
sudo sed -i 's/deb http:\/\/httpredir.debian.org\/debian\/ jessie main/deb http:\/\/httpredir.debian.org\/debian\/jessie main contrib/' /etc/apt/sources.list
sudo apt-get update -y && sudo apt-get install -y libgl1-mesa-glx libfontconfig1 libxslt1.1 libxtst6 libxxf86vm1 libgtk2.0-0 java-package
make-jpkg jdk-7u79-linux-x64.tar.gz
sudo dpkg -i oracle-java7-jdk_7u79_amd64.deb
```

#### Step 2. Install your PostgreSQL Database Server ####

こちらも **8.4.x.** というバージョン指定がある。
Java と同様に既に [サポート終了](https://www.postgresql.org/support/versioning/) しているため、
Debian のリポジトリでは配布されていない。
そのため、リポジトリを追加してからインストールする必要がある。

``` bash
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt/ $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
sudo apt-get update -y && sudo apt-get upgrade -y && sudo apt-get install -y postgresql-8.4
```

#### Step 3. Create your JIRA Database in PostgreSQL ####

Atlassian 製品はデータベースとして MySQL や PostgreSQL を利用出来る。
ただ、日本語や絵文字などマルチバイト文字を扱う場合は PostgresSQL を使うのが無難なようだ( [参考](https://confluence.atlassian.com/jirakb/jira-does-not-work-with-emoji-4-byte-characters-429919955.html) )。

個人的にあまり馴染みがないのだけれど、このドラゴンズレアでも PostgreSQL が指定されているので従っておく。

``` bash
sudo -u postgres createuser -S -d -r -P -E jirauser
# パスワード入力
sudo -u postgres createdb --owner jirauser --encoding utf8 jira
```

#### Step 4. Install JIRA ####

JIRA のバージョンも指定されている。 **6.3.15** を使う必要がある。

``` bash
wget https://www.atlassian.com/software/jira/downloads/binary/atlassian-jira-6.3.15-x64.bin
chmod a+x atlassian-jira-6.3.15-x64.bin
sudo ./atlassian-jira-6.3.15-x64.bin
```

JIRA にはインストーラーがついているので簡単。


### Step 5. Set Up JIRA, Step 6. Set up a Project and Create your JIRA Dashboard ###

あとはドキュメントに従って Web UI 上で操作を行えばよい。

もしかすると、以下のようなよく分からないエラーに遭遇するかもしれない。

```
let.ServletException: javax.servlet.jsp.JspTagException: Soy rendering failed for template '%s'.
説明 The server encountered an internal error that prevented it from fulfilling this request.

例外

org.apache.jasper.JasperException: javax.servlet.ServletException: javax.servlet.jsp.JspTagException: Soy rendering failed for template '%s'.
```

環境依存なのか、言語を日本語にしていた影響なのか分からないが、エラーが発生したらブラウザをリロードすれば大体直っていた。

### [Dragons Stage 2 - JIRA Add-Ons](https://confluence.atlassian.com/display/ATLAS/Dragons+Stage+2+-+JIRA+Add-Ons) ###

ドキュメントに従うだけで特に問題はないが、 Capture for JIRA を動かそうとしたときに以下のエラーが出ていた。

> Your Bonfire license has expired. Please visit My Atlassian to renew

ボンファイアとはなんのこったい、と思ったのだが、どうやら Capture for JIRA のことらしい。
インストール後のアクティベーションが正常に終わっていなかったようだった。
改めて管理画面から評価用ラインセンスを払い出してことなきを得た。

### [Dragons Stage 3 - Install Confluence](https://confluence.atlassian.com/display/ATLAS/Dragons+Stage+3+-+Install+Confluence) ###

#### Step 1. Create your Confluence Database in PostgreSQL ####

JIRA のときと同様に行う。

``` bash
sudo -u postgres createuser -S -d -r -P -E confuser
sudo -u postgres createdb --owner confuser --encoding utf8 confluence
```

#### Step 2. Install Confluence ####

インストールまでは以下の通り:

``` bash
wget https://www.atlassian.com/software/confluence/downloads/binary/atlassian-confluence-5.7.1-x64.bin
chmod a+x atlassian-confluence-5.7.1-x64.bin
sudo ./atlassian-confluence-5.7.1-x64.bin
```

この後、

> Because Confluence will be running on the same machine as JIRA (already installed), you need to ensure that the URL paths are different for Confluence and JIRA.

という理由で設定ファイルを一部いじる必要がある。
以下のように一旦 Confluence を停止、設定を修正して再起動する:

``` bash
sudo /opt/atlassian/confluence/bin/stop-confluence.sh
sudo sed -i 's/<Context path=""/<Context path="\/confluence"/' /opt/atlassian/confluence/conf/server.xml
sudo /opt/atlassian/confluence/bin/start-confluence.sh
```

#### Step 3 以降 ####

ドキュメントに従って進めれば問題ないはず。

### [Dragons Stage 4 \- Install Team Calendars in Confluence](https://confluence.atlassian.com/display/ATLAS/Dragons+Stage+4+-+Install+Team+Calendars+in+Confluence) ###

ドキュメントに従って Web UI を操作すれば :ok_hand: 。

### [Dragons Stage 5 \- Install FishEye and Crucible](https://confluence.atlassian.com/display/ATLAS/Dragons+Stage+5+-+Install+FishEye+and+Crucible) ###


#### Step 1. Install Mercurial ####

``` bash
sudo apt-get -y install mercurial
```

#### Step 2. Create your FishEye Database in PostgreSQL ####

これまでと同様に行う:

``` bash
sudo -u postgres createuser -S -d -r -P -E fishuser
sudo -u postgres createdb --owner fishuser --encoding utf8 fisheye
```

#### Step 3. Install FishEye and Crucible ####

インストーラーが無いので若干手順が複雑になる。

まず必要なファイルをダウンロードして展開する:

``` bash
wget https://www.atlassian.com/software/fisheye/downloads/binary/fisheye-3.7.0.zip
sudo unzip -d /opt/atlassian/ fisheye-3.7.0.zip
```

アプリケーションのデータディレクトリを作成する。
他のアプリケーションに合わせて `/var/atlassian/application-data/fecru` にする:

``` bash
sudo mkdir /var/atlassian/application-data/fecru
```

`FISHEYE_INST` という環境変数が上記のデータディレクトリを指すようにする:

``` bash
sudo sh -c 'echo "export FISHEYE_INST=/var/atlassian/application-data/fecru" > /etc/profile.d/fecru.sh'
source /etc/profile.d/fecru.sh
```

設定ファイルを一部修正しつつ配置する:

``` bash
sudo cp /opt/atlassian/fecru-3.7.0/config.xml /var/atlassian/application-data/fecru/
sudo sed -i 's/<web-server>/<web-server context="\/fisheye">/' /var/atlassian/application-data/fecru/config.xml
```

Fecru サービスを稼動させるためのユーザーを作成する:

``` bash
sudo useradd -m -c 'Atlassian FishEye/Crucible' fecru
```

権限を適切に修正し、サービスを立ち上げる:

``` bash
sudo chown fecru -R /opt/atlassian/fecru-3.7.0/
sudo chown fecru -R /var/atlassian/application-data/fecru/
nohup sudo -u fecru /opt/atlassian/fecru-3.7.0/bin/run.sh &
```

本来は systemd とかでちゃんとサービス化した方がいいと思う :worried:

#### Step 4 以降 ####

ドキュメントに従えば :ok_hand:

### [Dragons Stage 6 \- Get JIRA and FishEye Talking](https://confluence.atlassian.com/display/ATLAS/Dragons+Stage+6+-+Get+JIRA+and+FishEye+Talking) ###

[このコメント](https://confluence.atlassian.com/display/ATLAS/Dragons+Stage+6+-+Get+JIRA+and+FishEye+Talking?focusedCommentId=844238014#comment-844238014)
にあるように、恐らく FishEye の Site URL を設定してからでないと JIRA リンクが上手く動かないので、
まず FishEye 上で Administration -> Global Settings -> Server から Site URL を設定しておく。

あとはドキュメントの手順通り設定を行えばよい。

### [Dragons Stage 7 \- Get JIRA and Crucible Talking](https://confluence.atlassian.com/display/ATLAS/Dragons+Stage+7+-+Get+JIRA+and+Crucible+Talking) ###

ドキュメントの通りで :ok:

### [Dragons Stage 8 \- Install Bamboo](https://confluence.atlassian.com/display/ATLAS/Dragons+Stage+8+-+Install+Bamboo) ###

#### Step 1. Create your Bamboo Database in PostgreSQL ####

これまでと同様:

``` bash
sudo -u postgres createuser -S -d -r -P -E bamuser
sudo -u postgres createdb --owner bamuser --encoding utf8 bamboo
```

#### Step 2. Install Bamboo ####

Fecru と同様にこちらもインストーラーが無いので複雑な手順が必要になる:

``` bash
sudo -u postgres createuser -S -d -r -P -E bamuser
sudo -u postgres createdb --owner bamuser --encoding utf8 bamboo
wget https://www.atlassian.com/software/bamboo/downloads/binary/atlassian-bamboo-5.3.tar.gz
sudo useradd -m -c 'Atlassian Bamboo' bamboo
sudo tar -C /opt/atlassian -zxvf atlassian-bamboo-5.3.tar.gz
sudo chown bamboo -R /opt/atlassian/atlassian-bamboo-5.3
sudo sh -c 'echo "bamboo.home=/home/bamboo" > /opt/atlassian/atlassian-bamboo-5.3/atlassian-bamboo/WEB-INF/classes/bamboo-init.properties'
sudo sed -i 's/<Context path=""/<Context path="\/bamboo"/' /opt/atlassian/atlassian-bamboo-5.3/conf/server.xml
sudo -u bamboo /opt/atlassian/atlassian-bamboo-5.3/bin/start-bamboo.sh
```

#### Step 3. Set Up Bamboo ####

まず Maven 3 の環境を整える必要がある。
ドキュメントの指示通
り
[Atlassian Plugin SDK](https://developer.atlassian.com/display/DOCS/Set+up+the+Atlassian+Plugin+SDK+and+Build+a+Project)
をインストールする:

``` bash
sudo sh -c 'echo "deb https://sdkrepo.atlassian.com/debian/ stable contrib" > /etc/apt/sources.list.d/atlassian.list'
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys B07804338C015B73
sudo apt-get install -y apt-transport-https
sudo apt-get -y update
sudo apt-get install -y atlassian-plugin-sdk
```

あとはドキュメント通りに進めればよいが、
Maven Executable のパスを指定する箇所は
`/usr/share/atlassian-plugin-sdk-6.2.9/apache-maven-3.2.1` のように
指定しなければならない(ドキュメントには `/usr/local/Atlassian/atlassian-plugin-sdk/apache-maven` とあるがそこには無い)。

### [Dragons Stage 9 \- Bamboo Gadgets and JIRA Victory](https://confluence.atlassian.com/display/ATLAS/Dragons+Stage+9+-+Bamboo+Gadgets+and+JIRA+Victory) ###

ドキュメントの指示通り JIRA に Bamboo ガジェットを追加すれば完了。

**The Battle is Won, the Dragon is Slain!**

## Tシャツを要求する ##

ページの指示通りやると、クーポンコードがもらえるので、
それを使って注文すればいい。

自分の注文情報はこんな感じ:

<iframe src="https://drive.google.com/file/d/0B4XWl5W7tB7IcFRKdlhFTnpZV2s/preview" width="640" height="480"></iframe>

注文してから 1〜2 週間で届くらしい。
