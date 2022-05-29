---
title: "Ubuntu で Bluetooth マウスが使えない時にやるといいかもしれないこと"
author: ["yewton"]
date: 2022-05-29T23:28:00+09:00
mylastmod: 2022-05-29T23:29:12+09:00
slug: "ubuntu-bluetooth-troubleshoot"
tags: ["ubuntu", "bluetooth", "mouse"]
categories: ["雑記"]
draft: false
image:
  caption: Background image by <a href="https://pixabay.com/ja/users/0fjd125gk87-51581/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=517497">0fjd125gk87</a> from <a href="https://pixabay.com/ja/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=517497">Pixabay</a>
---

[以前購入した Dell Latitude 3420](/2021/10/11/dell-latitude-3420-byo-ubuntu/) ( Xubuntu 入り )をクラムシェルで使うときに、マウスが入り用になった。

元々 [ロジクール M337](https://amzn.to/3wZm0kh) を持っていたので、これを使えばいいと思っていたのだけれど、何故か検出されない。
具体的には `blueman-manager` から、マウスが見えない。

検索してみると [[SOLVED] Bluetooth Mouse Won't Connect after Reboot - Ubuntu 18.04 LTS](https://ubuntuforums.org/showthread.php?t=2390542) が見つかった。

理屈は分からないのだけれど、 [BlueZ](http://www.bluez.org/) の `bluetoothctl` を使わないと接続出来ない機器があるらしい。

```shell
bluetoothctl
```

で、対話モードになる。この状態で

```text
scan on
```

を実行すると、 `Bluetooth Mouse M336/M337/M535` として検出された
( この時点で `blueman-manager` の方でも表示されるようになっていた )。

そして、デバイスの MAC アドレスを指定して、

```text
pair XX:XX:XX:XX:XX:XX
connect XX:XX:XX:XX:XX:XX
trust XX:XX:XX:XX:XX:XX
```

と実行すれば、ペアリングと接続が完了し、次回以降はマウス側から再接続も出来るようになった。
