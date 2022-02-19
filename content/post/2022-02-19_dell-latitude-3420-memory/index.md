---
title: "Dell Latitude 3420 にメモリを増設する"
author: ["yewton"]
date: 2022-02-19T21:51:00+09:00
mylastmod: 2022-02-19T21:51:37+09:00
slug: "dell-latitude-3420-memory"
categories: ["買ったモノ"]
draft: false
---

[以前の記事](/2021/10/11/dell-latitude-3420-byo-ubuntu/) で書いた [Dell Latitude 3420](https://japancatalog.dell.com/pd/latitude-3420.html) だが、
普段使いに支障は無いものの、本格的に開発作業をしようと思うと
いささかメモリが心許ない( 8GiB )為、メモリを増設することにした。

今回購入したのは、 [Crucial 16GB DDR4-3200 SODIMM CT16G4SFRA32A](https://amzn.to/34T6Jr1) 。
[Crucial のページ](https://www.crucial.jp/compatible-upgrade-for/dell/latitude-14-(3420)) に Latitude 3420 との互換性が謳われていたのできっと安心。

[Dell の公式ページ](https://www.dell.com/support/kbdoc/ja-jp/000185381/) に、分解方法の解説がある為、これに従い分解していく。

まずはサービスモードに入る必要がある。

公式ページ記載の通りに、 `B` キーを押しながら電源投入でいい…のだが、
ずっと押しっぱなしにしていると連打している扱いになるらしく、
確認画面などをすっ飛ばして直ぐ様
ビープ音を鳴らしつつサービスモードに入ってしまう。

特に問題は無いと思うのだけれど、知らないとビックリするので注意。

`B` を押しながら電源を入れ、3〜5 秒程度で離すと、
上手くいくと以下のように `OWNER TAG` という文字列が表示される。

{{< figure src="2022-02-19_21-00-38_PXL_20220218_081810718.jpg" >}}

更に AC アダプタの接続有無に関わらず、以下のような注意文言が表示され…

> Please Remove AC Adapter
>
> Press Any Key To Continue...

{{< figure src="2022-02-19_21-02-53_PXL_20220218_081832378.jpg" >}}

最後に以下のような説明が出力され、
「カッカッ」とドキッとするようなビープ音が3回流れて電源が落ちる。

> System Ready For Service After 3 Short Beeps (OR Wait 2 Seconds).
>
> After The Service Is Completed,
> Plug In AC And Press And Hold The Power Button For 2 Seconds
> To Resume Normal Operations.
>
> Press Any Key To Continue...

{{< figure src="2022-02-19_21-03-43_PXL_20220218_081846897.jpg" >}}

この後、公式ページ記載の通りに底面のベースカバーのネジを外す。

そしてベースカバーを外せばいい…のだが、すんなりとはいかなかった。
どうやったら外せたのかよく分かっていないのだが、
おそらくヒンジ側中央部分の引っかかりをうまいこと外せれば、
あとは持ち上げるだけで外せる…ハズ。

上手く外せたら、以下の部分にメモリを差し込む。
元々 8GiB のが差さっているので、その反対側になる。

{{< figure src="2022-02-19_21-18-28_photo1.jpg" >}}

[メモリーの増設方法 | IODATA アイ・オー・データ機器](https://www.iodata.jp/product/memory/info/tips/#list2) 等を参考に、
斜め 35 度くらいに差し込み、奥までいったら下に押し込む。

カチッとハマッたら、再びベースカバーをネジ止めして、AC アダプタに接続する。

サービスモードに入るときの説明では、
AC アダプタ接続後に電源ボタンを押す的なことが書いてあったが、
自分の場合は接続後すぐに起動した。

起動後は以下のような画面が表示される。

> Alert! The amount of system memory has changed.

{{< figure src="2022-02-19_21-30-58_PXL_20220218_085238190.jpg" >}}

[Dell のサポート技術文書](https://www.dell.com/support/kbdoc/ja-jp/000137726/) によれば、無視して構わないとのこと。

> メモリを増設または取り外した場合、このメッセージは単なる通知として無視して構いません

以上で無事、既存の 8 GiB + 16 GiB で 24 GiB に RAM を増強出来た 🎉

{{< figure src="2022-02-19_21-41-36_screenshot.png" >}}
