---
title: "Chromebook Acer Spin 311 に Emacs を入れて幸せになる"
author: ["yewton"]
date: 2021-02-21T23:24:00+09:00
mylastmod: 2021-02-21T23:24:37+09:00
slug: "acer-spin-311-emacs"
tags: ["chromebook", "emacs", "acer"]
categories: ["買ったモノ"]
draft: false
---

<div class="ox-hugo-toc toc">
<div></div>

<div class="heading">&#30446;&#27425;</div>

- [Emacs のインストール](#emacs-のインストール)
- [(おまけ)git の credential helper を設定する](#おまけ--git-の-credential-helper-を設定する)
- [(おまけ)スクリーンショットの利用](#おまけ--スクリーンショットの利用)
- [(おまけ)arm 版 hugo-extended を作る](#おまけ--arm-版-hugo-extended-を作る)
- [終わりに](#終わりに)

</div>
<!--endtoc-->

[Chromebook Acer Spin 311](https://amzn.to/2ZBcsLU) を半ば衝動買いした。

当初の目的としては、電子書籍を [Moon+ Reader Pro](https://play.google.com/store/apps/details?id=com.flyersoft.moonreaderp) で読みつつ、
[Orgzly](https://play.google.com/store/apps/details?id=com.orgzly) でメモを取り、それを [Syncthing](https://play.google.com/store/apps/details?id=com.nutomic.syncthingandroid) で共有する、というような
Android でのユースケースを、キーボード付きのデバイスでやりたいな、というくらいだった。

実際購入してみたところ、 Linux (Debian) をシュッと立ち上げて使えるということだったので、それならばと Emacs を使えるようにしよう、と相成った。

この Linux は本当に Debian なので、実は Chromebook 特有の何かというのはあまりなかったのだけれど、備忘録として作業内容を記録しておく。


## Emacs のインストール {#emacs-のインストール}

Linux コンテナを立ち上げたらまずは更新をかける:

```sh
sudo apt update
sudo apt upgrade
```

日本語ロケールが設定されていないので、設定しておく:

```sh
sudo dpkg-reconfigure locales
```

Emacs や開発系のツールは新し目のを使いたいので、 nix で導入する。

こういう場合に snap や flatpak 等で入れると、
Emacs 内からリンクを開く際に既存のブラウザインスタンスが使われなかったり等の変な挙動をするので、 nix を使うようにしている
(参考: [Opening urls from emacs in firefox : r/emacs](https://www.reddit.com/r/emacs/comments/dj3abj/opening%5Furls%5Ffrom%5Femacs%5Fin%5Ffirefox/f4lctw7?utm%5Fsource=share&utm%5Fmedium=web2x&context=3))。

そして、現状の ChromeOS 特有の問題として、
nix のインストールに失敗する、という物がある(参考: [can't install nix-2.3.7 on ChromeOS linux container: operation not permitted mounting /proc · Issue #4107 · NixOS/nix](https://github.com/NixOS/nix/issues/4107))。ここに記載の通りワークアラウンドとして以下を実行する:

```sh
sudo umount /proc/{cpuinfo,diskstats,meminfo,stat,uptime}
```

これで nix のインストールが出来るようになる:

```sh
curl -L https://nixos.org/nix/install | sh
```

Emacs のインストールは以下の通り:

```sh
nix-env -i emacs-27.1
```

インストールしたアプリを ChromeOS 側から見えるように、以下のように設定する(参考: [Installing Nix on Crostini - NixOS Wiki](https://nixos.wiki/wiki/Installing%5FNix%5Fon%5FCrostini)):

```sh
mkdir -p ~/.config/systemd/user/cros-garcon.service.d/
cat > ~/.config/systemd/user/cros-garcon.service.d/override.conf <<EOF
[Service]
Environment="PATH=%h/.nix-profile/bin:/usr/local/sbin:/usr/local/bin:/usr/local/games:/usr/sbin:/usr/bin:/usr/games:/sbin:/bin"
Environment="XDG_DATA_DIRS=%h/.nix-profile/share:%h/.local/share:/usr/local/share:/usr/share"
EOF
```

これで一度再起動すれば、もう普通に使えるようになってしまう。


## (おまけ)git の credential helper を設定する {#おまけ--git-の-credential-helper-を設定する}

GitHub のプライベートリポジトリを利用する場合等には、パーソナルアクセストークンによる認証が必要になる。

Chromebook でも `gnome-keyring` を入れれば libsecret による認証ヘルパーを利用出来る。

```sh
sudo apt install gnome-keyring -y
(cd ~/.nix-profile/share/git/contrib/credential/libsecret/ && sudo make)
git config --file ~/.gitconfig.local credential.helper ~/.nix-profile/share/git/contrib/credential/libsecret/git-credential-libsecret
```

これで、認証が必要なリポジトリをクローンしようとすると、以下のようなダイアログが出てくるようになる:

{{< figure src="2021-02-21_22-11-22_screenshot.png" >}}


## (おまけ)スクリーンショットの利用 {#おまけ--スクリーンショットの利用}

上記のようなスクリーンショットは Chromebook 側で撮影したあとにクリップボードに送信することで、 Linux 側からも `xclip` 等で普通に使えるようになる。


## (おまけ)arm 版 hugo-extended を作る {#おまけ--arm-版-hugo-extended-を作る}

せっかく Emacs が使えるようになったので、ブログも Chromebook で書いてしまおうと思い、
Hugo をインストールしようとした所で引っ掛かってしまった。

現在利用しているテーマ( [Wowchemy](https://wowchemy.com/) )では、 Hugo の Extended 版が必要なのだが、
Spin 311 の CPU は arm であり、また Hugo 公式では arm 用の Extended バイナリを配布していない
(参考: [Extended Version for ARM - support - HUGO](https://discourse.gohugo.io/t/extended-version-for-arm/23762))。

そこで、 [Install Update Hugo Extended for ARM in Ubuntu | Apoorv Blog](https://apoorv.blog/posts/install-update-hugo-extended-for-arm.html) を参考にしつつ
(と言っても、 arm 版 go を入れて `go build --tags extended` しましょう、というだけだが…)、ソースからビルドすることで対応した。

なお、 go 自体のインストールは [asdf](https://github.com/asdf-vm/asdf) の [Go プラグイン](https://github.com/kennyp/asdf-golang) を利用した。


## 終わりに {#終わりに}

Moon+ Reader で電子書籍を読みつつ、 Emacs でメモを取り、
Android の Syncthing で同期する、という面白い環境が出来上がった。

ついでにブログ執筆環境も整えることが出来た。サクサク、というわけには行かないが、それほどストレスなく利用出来るし、キーボードも案外打ち易い。この記事自体も Chromebook で書いている。

なかなか良い買い物だったと思う。
