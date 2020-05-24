---
title: "MacBook Pro (Retina, 15-inch, Mid 2012) で macOS と Xubuntu 19.10 Eoan Ermine をデュアルブートする"
author: ["yewton"]
date: 2020-02-13T07:01:00+09:00
mylastmod: 2020-02-13T08:25:28+09:00
slug: "macbookpro10-xubuntu"
tags: ["macos", "ubuntu", "xubuntu", "xfce", "hidpi"]
categories: ["技術系"]
draft: false
image:
  caption: Background image by <a href="https://pixabay.com/users/bharathsiddamjetix37-3810891/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=1844603">Bharat Siddam</a> from <a href="https://pixabay.com/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=1844603">Pixabay</a>
---

<div class="ox-hugo-toc toc">
<div></div>

<div class="heading">&#30446;&#27425;</div>

- [前提](#前提)
- [パーティションの分割](#パーティションの分割)
- [起動用USBドライブの作成](#起動用usbドライブの作成)
- [Xubuntu インストール](#xubuntu-インストール)
- [Xubuntu 初期設定](#xubuntu-初期設定)
    - [キーテーマをEmacs風(macOS風)にする](#キーテーマをemacs風--macos風--にする)
    - [タッチパッドの設定(ナチュラルスクロール、水平スクロール)](#タッチパッドの設定--ナチュラルスクロール-水平スクロール)
    - [キーボードの設定(ファンクションキーの有効化)](#キーボードの設定--ファンクションキーの有効化)
    - [macOS Dock 風のランチャ(Plank)](#macos-dock-風のランチャ--plank)
    - [Alfred, Spotlight 風のランチャ(Albert)](#alfred-spotlight-風のランチャ--albert)
    - [Cica フォントの追加](#cica-フォントの追加)
    - [asdf のインストールと設定](#asdf-のインストールと設定)
    - [Emacs のインストールと設定](#emacs-のインストールと設定)
    - [ターミナル関連の設定](#ターミナル関連の設定)
    - [git の認証ヘルパ設定](#git-の認証ヘルパ設定)
    - [HiDPI への対応](#hidpi-への対応)
- [参考にしたリンク集](#参考にしたリンク集)

</div>
<!--endtoc-->


## 前提 {#前提}

-   MacBook Pro の種類は [15インチMacBook Pro Retinaディスプレイモデル：2.6GHz](https://support.apple.com/kb/SP653?locale=ja%5FJP)
    -   OS は [OS X El Capitan](https://support.apple.com/ja-jp/HT206886)
-   Xubuntu は [Xubuntu 19.10 Eoan Ermine](https://xubuntu.org/news/xubuntu-19-10-released/)
    -   目玉は `improved HiDPI support`


## パーティションの分割 {#パーティションの分割}

まず **[FileVault](https://support.apple.com/ja-jp/HT204837) が有効な状態ではパーティション操作が出来ない** ので、システム設定から解除する。
解除するだけで数時間かかるので気長に待つ。

FileVault の無効化が済んだら、 [Mac 用ディスクユーティリティ](https://support.apple.com/ja-jp/guide/disk-utility/welcome/mac) を使って Xubuntu インストール用のパーティションを用意する。

初期状態だとパーティション作成メニューが選択出来ないので、
[No partition scheme option when erasing a USB disk in MacOS High Sierra? - Ask Different](https://apple.stackexchange.com/questions/304131/no-partition-scheme-option-when-erasing-a-usb-disk-in-macos-high-sierra) を参考に
全てのデバイスを表示し、 **ボリュームではなくディスクを選択** する。

今回は単純に半分を mac に、もう半分を Xubuntu で使うように二等分した。
パーティション追加時の初期状態のまま。

このパーティション分割も数時間かかるので気長に待つ。


## 起動用USBドライブの作成 {#起動用usbドライブの作成}

`xubuntu-19.10-desktop-amd64.iso` を [Xubuntu 公式サイト](https://xubuntu.org/download) からダウンロードする。

[Create a bootable USB stick on macOS | Ubuntu](https://ubuntu.com/tutorials/tutorial-create-a-usb-stick-on-macos) を参考に、 [Etcher](https://www.balena.io/etcher/) で上記の OS イメージを焼く。

用意したのは容量 2GB という骨董品レベルの USB スティックドライブだったが、問題なく動いた。


## Xubuntu インストール {#xubuntu-インストール}

[Startup Manager](https://support.apple.com/ja-jp/HT202796) を起動し、上記の起動USBドライブを挿入する。

すると下の画像のように黄色い `EFI Boot` という名前のドライブが出現するので、
これを選択する。

何故か二つ出現することがあるのだが、どちらを選んでも大丈夫そうだった。

{{< figure src="2020-02-13_04-45-56_startup-manager.jpg" >}}

**初期状態では Wi-Fi が使えない** はずなので、 [Rankie の有線LAN アダプタ](https://amzn.to/2OPxvW3) など
ドライバのインストール不要な適当な USB イーサネットアダプタを用意しておく必要がある。

インストール時に
`グラフィックスとWi-Fiハードウェアと追加のメディアフォーマットのサードパーティ製ソフトウェアをインストールする` をチェックしておくと、
初回起動時から Wi-Fi が使えるようになる模様。

いずれにしても有線環境は必要。

[SwapFaq - Community Help Wiki](https://help.ubuntu.com/community/SwapFaq) を参考に以下のように `swap` パーティションだけ用意したが、
別に要らなかったかもしれない。あとから [SwapFile](https://wiki.archlinux.org/index.php/Swap#Swap%5Ffile) を追加することも出来るようだし。

`ブートローダをインストールするデバイス` は Xubuntu のルートパーティションを選択した。

{{< figure src="2020-02-13_04-47-05_IMG_20200211_045148.jpg" >}}

インストール完了後はデフォルトで Xubuntu が起動するようになる。

変えたくなったら再び Startup Manager を起動すればよい。


## Xubuntu 初期設定 {#xubuntu-初期設定}


### キーテーマをEmacs風(macOS風)にする {#キーテーマをemacs風--macos風--にする}

```sh
xfconf-query -c xsettings -p /Gtk/KeyThemeName -s Emacs
```


### タッチパッドの設定(ナチュラルスクロール、水平スクロール) {#タッチパッドの設定--ナチュラルスクロール-水平スクロール}

以下のようなシェルスクリプトを作成し、 `セッションと起動` メニューからログイン時に実行するように設定する。

```sh
#!/usr/bin/env bash

synclient VertScrollDelta=-$(synclient | grep VertScrollDelta | awk '{print $3}')
synclient HorizTwoFingerScroll=1
synclient HorizScrollDelta=-$(synclient | grep HorizScrollDelta | awk '{print $3}')
```

<div class="src-block-caption">
  <span class="src-block-number">ソースコード 1</span>:
  <code>fix_scroll.sh</code>
</div>


### キーボードの設定(ファンクションキーの有効化) {#キーボードの設定--ファンクションキーの有効化}

```sh
echo 'options hid_apple fnmode=2' | sudo tee -a /etc/modprobe.d/hid_apple.conf
```


### macOS Dock 風のランチャ(Plank) {#macos-dock-風のランチャ--plank}

シンプルな Dock 風のランチャ [Plank](https://launchpad.net/plank) をインストールし、 `セッションと起動` メニューからログイン時に実行するように設定する。

```sh
sudo apt install plank
```


### Alfred, Spotlight 風のランチャ(Albert) {#alfred-spotlight-風のランチャ--albert}

[Alfred](https://www.alfredapp.com/) や [Spotlight](https://support.apple.com/ja-jp/HT204014) のような使い心地のランチャ [Albert](https://albertlauncher.github.io/) をインストールし、 `セッションと起動` メニューからログイン時に実行するように設定する。

```sh
curl https://build.opensuse.org/projects/home:manuelschneid3r/public_key | sudo apt-key add -
echo 'deb http://download.opensuse.org/repositories/home:/manuelschneid3r/xUbuntu_19.10/ /' | sudo tee /etc/apt/sources.list.d/home:manuelschneid3r.list
apt update
apt install albert
```


### Cica フォントの追加 {#cica-フォントの追加}

[miiton/Cica: プログラミング用日本語等幅フォント Cica(シカ)](https://github.com/miiton/Cica) をインストールする。

```sh
wget https://github.com/miiton/Cica/releases/download/v5.0.1/Cica_v5.0.1_with_emoji.zip
unzip Cica_v5.0.1_with_emoji.zip
sudo mv *.ttf /usr/local/share/fonts/
sudo fc-cache -fv
```


### asdf のインストールと設定 {#asdf-のインストールと設定}

[asdf](https://asdf-vm.com/#/core-manage-asdf-vm) をインストールし、プラグインに必要なパッケージをインストールする。

```sh
sudo apt install \
  automake autoconf libreadline-dev \
  libncurses-dev libssl-dev libyaml-dev \
  libxslt-dev libffi-dev libtool unixodbc-dev \
  unzip curl \
  libbz2-dev libreadline-dev libsqlite3-dev -y
```


### Emacs のインストールと設定 {#emacs-のインストールと設定}

[GNU Emacs](https://www.gnu.org/software/emacs/) をインスールし、利用するパッケージをインストールする。

```sh
sudo apt install build-essential texinfo aspell ripgrep cmigemo -y
sudo snap install emacs --classic
```


### ターミナル関連の設定 {#ターミナル関連の設定}

```sh
wget -qO - https://apt.thoughtbot.com/thoughtbot.gpg.key | sudo apt-key add -
echo "deb https://apt.thoughtbot.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/thoughtbot.list
sudo apt-get update
sudo apt install rcm curl fzf fasd tmux powerline -y
curl -sL --proto-redir -all,https https://raw.githubusercontent.com/zplug/installer/master/installer.zsh | zsh
```

fzf は `/usr/share/doc/fzf/README.Debian` にある指示に従う。


### git の認証ヘルパ設定 {#git-の認証ヘルパ設定}

```sh
sudo apt-get install libsecret-1-0 libsecret-1-dev
(cd /usr/share/doc/git/contrib/credential/libsecret/ && sudo make)
echo -e "\n[credential]\n helper = /usr/share/doc/git/contrib/credential/libsecret/git-credential-libsecret\n" >> ~/.gitconfig.local
```


### HiDPI への対応 {#hidpi-への対応}


#### 全般 {#全般}

`外観` メニューで `設定` タブの `ウィンドウ拡大縮小` を `2倍` に設定する。

{{< figure src="2020-02-13_06-59-55_Screenshot_2020-02-13_06-59-30.png" >}}

`/etc/environment` に以下を追記する。

```sh
GDK_SCALE=2
QT_AUTO_SCREEN_SCALE_FACTOR=0
QT_SCALE_FACTOR=2
```


#### Emacs {#emacs}

フォントサイズが小さいままなので、サイズ指定を倍にすることで暫定対応。

ただ、画像やモードラインの高さなどが小さいままなので、対応検討中。


#### GIMP {#gimp}

テーマ設定でフォントを大きくするしかない。

```sh
cp -r /usr/share/gimp/2.0/themes/Dark ~/.config/GIMP/2.10/themes/MyDark
```

等で元になるテーマをコピーし、以下のような指定を追記(コメントアウトを解除)する。

```conf
gtk-font-name = "Noto Sans Regular 18"
font_name = "Noto Sans Regular 18"
```


## 参考にしたリンク集 {#参考にしたリンク集}

-   [No partition scheme option when erasing a USB disk in MacOS High Sierra? - Ask Different](https://apple.stackexchange.com/questions/304131/no-partition-scheme-option-when-erasing-a-usb-disk-in-macos-high-sierra)
-   [MacBook AirをXubuntuとのデュアルブートにした。 | b-shock. Fortress](https://blog.b-shock.org/2018/03/03/Xubuntu-MacBook-Air/)
-   [Create a bootable USB stick on macOS | Ubuntu](https://ubuntu.com/tutorials/tutorial-create-a-usb-stick-on-macos)
-   [HiDPI - ArchWiki](https://wiki.archlinux.org/index.php/HiDPI)
-   [MacBookPro10,x - ArchWiki](https://wiki.archlinux.org/index.php/MacBookPro10,x)
-   [macでUSB起動したxubuntuでWiFiを使えた！ - そよ風ブログ - esperas! エスペラントの世界](http://esperas.info/index.php?QBlog-20180312-2)
-   [SwapFaq - Community Help Wiki](https://help.ubuntu.com/community/SwapFaq)
-   [How to Install and Dual-Boot Ubuntu on Mac - Make Tech Easier](https://www.maketecheasier.com/install-dual-boot-ubuntu-mac/)
-   [How do you remap a key to the Caps Lock key in Xubuntu? - Ask Ubuntu](https://askubuntu.com/questions/149971/how-do-you-remap-a-key-to-the-caps-lock-key-in-xubuntu)
-   [UbuntuUpdates - PPA: Google Chrome](https://www.ubuntuupdates.org/ppa/google%5Fchrome)
-   [xubuntu - How to enable natural scrolling in xfce4? - Ask Ubuntu](https://askubuntu.com/questions/690512/how-to-enable-natural-scrolling-in-xfce4)
-   [11193 – GTK3 apps don't understand natural scrolling](https://bugzilla.xfce.org/show%5Fbug.cgi?id=11193)
-   [touchpad - How do I enable horizontal scroll on lubuntu desktop - Ask Ubuntu](https://askubuntu.com/questions/440670/how-do-i-enable-horizontal-scroll-on-lubuntu-desktop)
-   [apps:terminal:dropdown [Xfce Docs]​](https://docs.xfce.org/apps/terminal/dropdown)
-   [Xubuntu: \`xdg-user-dirs-gtk-update\`でユーザーディレクトリが英語名になったけど、なぜか\`デスクトップ\`ディレクトリが英語にならない - Qiita](https://qiita.com/yuji38kwmt/items/cffc3507e0cbd0b76454)
-   [Ubuntu 19.04 package · Issue #793 · albertlauncher/albert](https://github.com/albertlauncher/albert/issues/793)
-   [On an Apple Keyboard under Linux, how do I make the Function keys work without the fn modifier key? - Unix & Linux Stack Exchange](https://unix.stackexchange.com/questions/121395/on-an-apple-keyboard-under-linux-how-do-i-make-the-function-keys-work-without-t)
