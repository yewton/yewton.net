---
title: "スーパーマリオ64 Behind Camera Anywhere とは"
author: ["yewton"]
date: 2021-06-07T00:30:00+09:00
mylastmod: 2021-06-07T00:30:21+09:00
slug: "mario64-behind-camera-anywhere"
categories: ["雑記"]
draft: false
image:
  caption: Photo by <a href="https://unsplash.com/@claudiolcastro?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Cláudio Luiz Castro</a> on <a href="https://unsplash.com/s/photos/mario-64?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>
---

## 背景 {#背景}

pannenkoek2012 氏( [メインチャンネル](https://www.youtube.com/user/pannenkoek2012) , [セカンダリチャンネル](https://www.youtube.com/user/pannenkeok2012) ) の上げているマリオ 64 動画
( 極力 A ボタンを押さないプレイ、いわゆる A Button Challenge, ABC の方 )が好きで、ちょくちょく動画を拝見している。

さる 2021 年 5 月 3 日に新たな動画が投稿されており、何やら新たなテクニックを活用されているようだった:

{{< youtube BMwzx6NCUOs >}}

説明文から参照されている先を見ると、 **Behind Camera Anywhere** というモノらしい:

{{< youtube C38He-OUAYk >}}

ABC の新たな展開に備え、このテクニックを理解しようと考えた。

<div class="ox-hugo-toc toc">
<div></div>

<div class="heading">&#30446;&#27425;</div>

- [背景](#背景)
- [前提知識](#前提知識)
- [Behind Camera Anywhere](#behind-camera-anywhere)
- [終わりに](#終わりに)
- [リンク集](#リンク集)

</div>
<!--endtoc-->


## 前提知識 {#前提知識}

Behind Camera Anywhere は、 HOLP 固定手法の一つである。

HOLP については [スーパーマリオ64学入門【アドベントカレンダー2018　32日目】 | 東京工業大学デジタル創作同好会traP](https://trap.jp/post/555/)  が詳しい。

簡単に説明しておくと、まず HOLP とは Held Object's Last Place 、
つまりマリオが手に持ったオブジェクトの最終位置のこと。

通常はマリオの移動と共に更新され続けるが、
何らかの理由でマリオがレンダリングされていない場合等には
HOLP が更新されずに以前の値のままになるという性質があり、
この性質を利用することを **HOLP 固定** と呼ぶ。

pannenkoek2012 氏による以下の動画でも解説されている:

{{< youtube pHihyGlYfSw >}}

また、 **HOLP はワールドをまたいでもリセットされない** という性質もある。

この性質と、オブジェクトの消滅と保持をほぼ同時に発生させることで、
ワールド中にロードされるほぼ任意のオブジェクト(のクローン)をマリオに持たせられる技、
いわゆる **無を取得** との合わせ技が ABC ではよく使われる。

例えば、ポールから降りる際の A ボタン入力を回避する為に、
降りたい地点に敵オブジェクトを配置することでヒットバックを発生させ、無理矢理降りる、等である。


## Behind Camera Anywhere {#behind-camera-anywhere}

マリオを画面外に追いやることでレンダリングを回避し、
結果 HOLP を固定出来ること自体は [先程の動画](https://www.youtube.com/watch?v=pHihyGlYfSw) の公開時点(2016 年)で発見されていたものの、
その条件である「マリオを画面外に追いやる」手法が確立されていなかった。

今回発見された Behind Camera Anywhere ではその名の通り、どこでもカメラの後ろに回れる、
つまり **(ほぼ)任意の場所でマリオを画面外に追いやれる** ようになったのである。

これまでの HOLP 固定は、 [先程の動画](https://www.youtube.com/watch?v=pHihyGlYfSw) でも紹介されているポーズバッファリングを用いたヒットスタン
(Pause Buffered Hitstun, PBH) を利用する方法が主流だった。

PBH を簡単に説明すると、被弾時の点滅状態に於いて、
ゲーム内タイマーの値が偶数の場合のみマリオがレンダリングされること、
及びポーズ中でも該当タイマーの値は進み続けることを利用し、
適当な間隔でポーズを連打することで、
被弾無敵時間が持続する限り一切のレンダリングを避け、
結果として HOLP を固定する、という技である。

PBH を利用するには必ず被弾する為の敵が必要になるが、
一方で今回の Behind Camera Anywhere ではそれが不要になる。

とはいえ [発見元の動画](https://www.youtube.com/watch?v=C38He-OUAYk) 説明文にもある通り、
画面外を維持したまま出来る操作というのは限られており、
必ずしも PBH の代替手段とはならないのだそう。

この手法にもそれなりの制限があるということではあるが、
新たな HOLP 固定手法として今後活用される機会は増えていくかもしれない。


## 終わりに {#終わりに}

筆者自身はプレイするでも、解析するでもなく、
氏の解説動画を観て分かった気になっているだけである。

その為、本記事の内容には誤りがある可能性もある。

是非、各位自身の目で、氏による懇切丁寧な解説動画を確認し、
奥深いスーパーマリオ 64 A ボタンチャレンジの世界に飛び込んで頂きたい。


## リンク集 {#リンク集}

-   [pannenkoek2012 氏のメインチャンネル](https://www.youtube.com/user/pannenkoek2012)
-   [pannenkoek2012 氏のセカンダリチャンネル](https://www.youtube.com/user/pannenkeok2012)
-   [スーパーマリオ64学入門【アドベントカレンダー2018　32日目】 | 東京工業大学デジタル創作同好会traP](https://trap.jp/post/555/)
