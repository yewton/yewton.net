---
title: "Dell Latitude 3420 の SSD をより大容量の SSD に Clonezilla でクローンし換装する"
author: ["yewton"]
date: 2025-01-06T01:05:00+09:00
mylastmod: 2025-01-06T01:09:21+09:00
slug: "dell-latitude-3420-storage"
tags: ["dell", "latitude", "clonezilla"]
categories: ["買ったモノ"]
draft: false
---

<div class="ox-hugo-toc toc">
<div></div>

<div class="heading">&#30446;&#27425;</div>

- [はじめに](#はじめに)
- [Clonezilla による SSD クローン](#clonezilla-による-ssd-クローン)
- [SSD の換装](#ssd-の換装)
- [メモリの換装](#メモリの換装)
- [動作確認](#動作確認)

</div>
<!--endtoc-->


## はじめに {#はじめに}

[以前の記事](/2022/02/19/dell-latitude-3420-memory/) では [Dell Latitude 3420](https://japancatalog.dell.com/pd/latitude-3420.html) にメモリを増設したが、今回は SSD を換装した。

Docker イメージやら IDE やら何やらで 256 GiB でも心許なくなってきた為、 1 TiB の SSD に換装した。

せっかくなので、ついでにメモリも追加で購入し、 8 GiB + 16 GiB の 24 GiB から 16 GiB × 2 で 32 GiB に。

今回購入した SSD は、 [キオクシア ( KIOXIA ) の EXCERIA PRO NVMe™ M.2 Type 2280-S2-M SSD 1TB](https://amzn.to/3DHMFZP) 。
[Dell Latitude 3420 Notebook Teardown removal guide for customer replaceable units (CRUS) | Dell US](https://www.dell.com/support/kbdoc/en-us/000185381/dell-latitude-3420-notebook-teardown-removal-guide-for-customer-replaceable-units-crus?lwp=rt) に
M.2 SSD 2280 card slot の記載があるので問題ないハズ。

この SSD を [ロジテック ( Logitec ) の SSD ケース LGB-PNV02UC](https://amzn.to/40jZ5jc) にセットして、データのクローンを行った。

メモリは [Crucial 16GB DDR4-3200 SODIMM D4N3200CM-16GQ](https://amzn.to/405kHhX) を購入。


## Clonezilla による SSD クローン {#clonezilla-による-ssd-クローン}

クローンには [Clonezilla](https://clonezilla.org/) を利用した。

まずは [Clonezilla Live on USB](https://clonezilla.org///liveusb.php) に従って Live USB を作成する。

今回はたまたま余っていた [エレコム 外付けポータブルSSD ESD-ED0120GBK 120GB](https://amzn.to/3PJ2Fh3) を利用したが、  500 MiB あれば何でもよいハズ。

手順に従い、 FAT でフォーマットした後に `unzip` で配置して完成。

この Live USB と、クローン先の SSD を収めたケースを接続した上で、 [Latitude 3420 サービス マニュアル | Dell 日本](https://www.dell.com/support/manuals/ja-jp/latitude-14-3420-laptop/latitude%5F3420%5Fcyborg%5Fsm/%E3%83%96%E3%83%BC%E3%83%88%E3%83%A1%E3%83%8B%E3%83%A5%E3%83%BC?guid=guid-493feb0b-5f68-470b-83ef-b8a975eb6e9d&lang=ja-jp#:~:text=%E3%82%B7%E3%82%B9%E3%83%86%E3%83%A0%E3%81%AE%E6%9C%89%E5%8A%B9%E3%81%AA%E8%B5%B7%E5%8B%95,%E3%82%92%E6%8A%BC%E3%81%97%E3%81%BE%E3%81%99%E3%80%82) に従い、起動後 Dell のロゴが表示されたときに `<F12>` を押してブートメニューに入り、 Clonezilla の Live USB を選択する。

その後は [Clonezilla Live Doc の Disk to disk clone のページ](https://clonezilla.org///fine-print-live-doc.php?path=clonezilla-live/doc/03%5FDisk%5Fto%5Fdisk%5Fclone) に従っていけばよい。

{{< figure src="2025-01-05_23-51-16_screenshot.png" caption="&#22259;1:  Start Clonezilla を選択する" >}}

{{< figure src="2025-01-05_23-52-43_screenshot.png" caption="&#22259;2:  device-device を選択する" >}}

元々単一のパーティションにしていたので、今回は [Clonezilla - Advanced Mode](https://clonezilla.org/clonezilla-live/doc/03%5FDisk%5Fto%5Fdisk%5Fclone/advanced/05-advanced-param.php) に従いパーティションも含めてよしなに拡張するオプションを有効化する為、 `Expert` を選択する。

{{< figure src="2025-01-05_23-57-51_screenshot.png" caption="&#22259;3:  Expert を選択する" >}}

{{< figure src="2025-01-05_23-58-30_screenshot.png" caption="&#22259;4:  disk\_to\_local\_disk を選択する" >}}

{{< figure src="2025-01-06_00-04-16_screenshot.png" caption="&#22259;5:  クローン元のディスクを選択する" >}}

{{< figure src="2025-01-06_00-10-00_screenshot.png" caption="&#22259;6:  クローン先のディスクを選択する(元のデータは全消去される)" >}}

{{< figure src="2025-01-06_00-11-37_screenshot.png" caption="&#22259;7:  advanced extra parameters はデフォルトのまま" >}}

{{< figure src="2025-01-06_00-12-26_screenshot.png" caption="&#22259;8:  ファイルシステムのチェックはスキップ(デフォルト)" >}}

{{< figure src="2025-01-06_00-13-22_screenshot.png" caption="&#22259;9:  ここで -k1 Create partition table proportionally を選択する" >}}

{{< figure src="2025-01-06_00-14-18_screenshot.png" caption="&#22259;10:  終了後の挙動はその時に選択することにして、進める" >}}

{{< figure src="2025-01-06_00-15-15_screenshot.png" caption="&#22259;11:  とても熱心に確認してくれるので、覚悟を決めて y する" >}}

10 分程で完了するので、ここで電源オフ。

続けて、クローンした SSD に換装する。


## SSD の換装 {#ssd-の換装}

[以前の記事](/2022/02/19/dell-latitude-3420-memory/) 同様にサービスモードに入り、ベースカバーを外して作業していく
( 今回気づいた点は以前の記事側に注記を入れた )。

元々は M.2 type2242 が使われていた為、以下の部分でネジ止めされていた。

{{< figure src="2025-01-06_00-32-06_screenshot.png" caption="&#22259;12:  元々ネジ止めされていた部分" >}}

換装するのは M.2 type2280 の為、この部品自体を移動する必要があった。

{{< figure src="2025-01-06_00-36-59_screenshot.png" caption="&#22259;13:  ネジを外して固定金具自体を移動する" >}}

この後、 SSD を接続して金具に固定し、 SSD の換装は完了。


## メモリの換装 {#メモリの換装}

ついでの作業だが記録として。

取り付けは [以前の記事](/2022/02/19/dell-latitude-3420-memory/) 同様だが、今回は元々入っていた 8 GiB メモリと交換する必要がある為、まず取り外す必要がある。

以下の固定金具部分を外側に押し広げるような感じにすると、バインッと外れた。

{{< figure src="2025-01-06_00-42-26_screenshot.png" caption="&#22259;14:  固定金具。右側は隠れてしまっているが同じ箇所にある。" >}}

外したあとは、 [以前の記事](/2022/02/19/dell-latitude-3420-memory/) 同様に新しいメモリをセットして完了。

{{< figure src="2025-01-06_00-50-01_screenshot.png" caption="&#22259;15:  追加分。こちら側は向きを逆にする必要があった。" >}}


## 動作確認 {#動作確認}

SSD、メモリのセットが完了したので、ベースカバーを閉じて通電、起動を確認する。

今回は以前と異なり、電源投入後に数分間無反応でヒヤヒヤしたが、その後問題無く起動し、追加のストレージ・RAM も問題無く認識された 🎉

{{< figure src="2025-01-06_01-02-47_screenshot.png" caption="&#22259;16:  Memory: 31.10 GiB 、 Disk (/): 907.06 GiB となっている" >}}
