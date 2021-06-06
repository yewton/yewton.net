---
title: "スーパーマリオ64 Behind Camera Anywhere とは"
author: ["yewton"]
date: 2021-06-06
mylastmod: 2021-06-06T23:16:04+09:00
slug: "mario64-behind-camera-anywhere"
categories: ["未分類"]
draft: true
image:
  caption: Photo by <a href="https://unsplash.com/@claudiolcastro?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Cláudio Luiz Castro</a> on <a href="https://unsplash.com/s/photos/mario-64?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>
---

## 背景 {#背景}

pannenkoek2012 氏( [メインチャンネル](https://www.youtube.com/user/pannenkoek2012) , [セカンダリチャンネル](https://www.youtube.com/user/pannenkeok2012) ) の上げているマリオ 64 動画
( 極力 A ボタンを押さないプレイ、いわゆる A Button Challenge, ABC の方 )が好きで、ちょくちょく動画を拝見しているのだが、さる 2021 年 5 月 3 日に新たな動画が投稿されており、何やら新たな手法を活用されているようだった:

{{< youtube BMwzx6NCUOs >}}

説明文から参照されている先を見ると、 **Behind Camera Anywhere** というモノらしい:

{{< youtube C38He-OUAYk >}}


## 前提知識 {#前提知識}

Behind Camera Anywhere は、 HOLP 固定手法の一つである。

HOLP については [スーパーマリオ64学入門【アドベントカレンダー2018　32日目】 | 東京工業大学デジタル創作同好会traP](https://trap.jp/post/555/)  が詳しい。

簡単に説明しておくと、まず HOLP とは Held Object's Last Place 、つまりマリオが手に持ったオブジェクトの最終位置のこと。

通常はマリオの移動と共に更新され続けるが、何らかの理由でマリオがレンダリングされていない場合等には
HOLP が更新されずに以前の値のままになるという性質があり、この性質を利用することを **HOLP 固定** と呼ぶ。

pannenkoek2012 氏による以下の動画でも解説されている:

{{< youtube pHihyGlYfSw >}}

また、 HOLP はワールドをまたいでもリセットされないという性質もあり、この性質と、オブジェクトのアンロードと保持をほぼ同時に行うことで、ワールド中にロードされるほぼ任意のオブジェクトをマリオに持たせられる技、いわゆる **無を取得** との組み合わせが ABC ではよく使われる。

例えば、ポールから降りる際の A ボタン入力を回避する為に、降りたい地点に敵を配置することでヒットバックを発生させて無理矢理降りる、等である。
