+++
categories = [
  "技術系",
]
tags = [
  "Android",
]
slug = "grouper-beanstalk"
banner = "banners/mb.jpg"
date = "2017-04-09T21:55:31+09:00"
title = "Nexus 7 2012 (grouper) に Beanstalk と ParrotMod を入れて蘇生する"

+++

最近はじめて [Fire タブレット](http://amzn.to/2nXsSKh) をキャンペーンで購入して、
こういう本読むだけとか書い物するだけとかの単機能なタブレットも結構いいな、と思った。
そして、うちに使ってない [Nexus 7](http://amzn.to/2oSEwJK) があることを思い出した。

この Nexus 7 、どうして使っていなかったかというと、
Lollipop にアップデートしてしまったから。
Nexus 7 2012 は Lollipop にアップデートしてしまうと、
動作が重過ぎて使い物にならなくなる。

どうせ使いものにならないなら、ということで
[SlimKat](http://www.slimroms.net/) や
[PureNexus](https://forum.xda-developers.com/nexus-7/development/unofficial-pure-nexus-project-layers-t3243943) を
焼いて使ってみたりもしたが、
前者は軽いけど見た目や使用感がちょっと微妙( 4.4 ベースだし…)で、
後者は使っていくうちにやっぱり重くなっていって駄目だった。

今回 Fire を買ってあらためて 7 インチの Android タブレットが欲しくなってしまったので、
もう一度蘇生を試みることにした。

そして [Reddit のこのスレ](https://www.reddit.com/r/Nexus7/comments/3h0oxg/recommended_fastlightweight_rom_for_2012_n7/) を満つけ、
そこから [Speed up the Nexus 7 with F2FS and SlimKat](https://teknovenus.com/speed-up-nexus-7-f2fs-slimkat-ghost/) という記事に辿り着いた。

元々は SlimKat と F2FS の記事だったようだが、2017 年に更新があり、
[Beanstalk 6\.0\.1](https://forum.xda-developers.com/nexus-7/development/rom-beanstalk-rom-t3312870) と
[ParrotMod](https://forum.xda-developers.com/nexus-7/orig-development/parrotmod-speed-2012-nexus-7-emmc-fix-t3300416) が
勧められていた(執筆者自身は試していないようだったが)。

## Beanstalk ##

CyanogenMod 13 (Marshmallow) をベースに作られたカスタム Rom 。
[フォーラム](https://forum.xda-developers.com/galaxy-s3/development/rom-t3370186) の記述によると、

> Smoother than Butter  
> Friendly Battery  
> Tons of Features  
> DirtyUnicorns Features  
> Always Up-to-date  
> 100% Build from Source

…らしい。

## ParrotMod ##

Nexus 7 2012 をすげーいい感じにしてくれるやつ。

> FLASH MEMORY SPEED INCREASE! up to 4x better performance WITHOUT F2FS OR DYNAMIC FSYNC

という速度の向上を筆頭に、様々な最適化を行って快適にしてくれるようだ。

## 結果 ##

Amazon Kindle , YouTube 、Feedly , 1Password あたりを主に動かしているが、
アプリ起動時にたまに固まることがあるものの、大抵一度タスクキルして起動し直せば問題なく動作している。
娯楽用途としては十分な性能になった。

もし、使わなくなった Nexus 7 2012 がご家庭に眠っているのなら、
Beanstalk と ParrotMod を試してみる価値はアリだと思う。
