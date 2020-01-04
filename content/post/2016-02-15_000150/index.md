+++
banner = "banners/dvvb.jpg"
categories = ["技術系"]
date = "2016-02-19T00:00:00+09:00"
slug = "save-storage-space-by-moving-vm-related-files-to-ex-storage"
tags = ["mac", "virtualbox", "docker", "vagrant"]
title = "VM関連ファイルを外部ストレージに保存して空き容量を確保する"

+++

## 何かと容量を食うVM関連ファイル ##

DockerやらVagrantやらで作業していると、いつのまにかディスク容量が逼迫していることがありますよね。

自分も128GB SSDのMacBook Proを使っているので、結構いっぱいいっぱいでした。

そこで拡張ストレージを用意して、容量を食いがちなVM関連のファイルを移動することにしました。

### (参考)Mac用の拡張ストレージ ###

USBで外付けするようなのだと持ち運びに不便なので、
<a rel="nofollow" href="http://www.amazon.co.jp/gp/product/B00TTFOJ4A/ref=as_li_ss_tl?ie=UTF8&camp=247&creative=7399&creativeASIN=B00TTFOJ4A&linkCode=as2&tag=yewton-22">iSlice Pro</a><img src="http://ir-jp.amazon-adsystem.com/e/ir?t=yewton-22&l=as2&o=9&a=B00TTFOJ4A" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" />
のような本体のSDカードスロットに差せるようなタイプを個人的には使っています。
iSliceの場合はただのアダプタなので別途128GBのmicroSDカードを調達する必要がありますが、ストレージと一体になっているモノよりは若干割安です。

<iframe src="https://rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=yewton-22&o=9&p=8&l=as4&m=amazon&f=ifr&ref=ss_til&asins=B00TTFOJ4A" style="width:120px;height:240px;" scrolling="no" marginwidth="0" marginheight="0" frameborder="0"></iframe>

## Vagrant ##

Vagrant用のBoxファイルとVMイメージの保存場所を変える際には、以下の記事が参考になります:

[MacBookAir の容量がきついので Vagrant 環境を外付けに移した話 – モンキーレンチ](http://2inc.org/blog/2014/06/28/4311/ "MacBookAir の容量がきついので Vagrant 環境を外付けに移した話 – モンキーレンチ")

上記の記事を参考に諸々のファイルの移動、VB上の設定を済ませたら、
以下のようなコマンドを `.bashrc` やら `.zshenv` やらに書いておけば大丈夫です:

`export VAGRANT_HOME=/Volumes/data/.vagrant.d`

## Docker Machine ##

Docker Machine用のファイルもデカいので移動させたいです。

Docker Machineでは `MACHINE_STORAGE_PATH` という環境変数を参照しています。
デフォルトは `~/.docker/machine` です。

Vagrantの場合と同じように既存のファイルを新しい場所に移動し、
VB上で除去→追加の手順を踏みます。
そして以下のようなコマンドで環境変数をセットします:

`export MACHINE_STORAGE_PATH=/Volumes/data/.docker/machine`

これも `.bashrc` やらに書いておきましょう。


## 注意: 外部ストレージのフォーマット ##

自分の場合、SDカードが元々フォーマットされていて、
差すだけで既に利用できたので、フォーマットについては特に気にせず移行作業をしてしまいました。

ところが、移行後に `'docker-machine` が謎のエラーで使えなくなってしまいました。

`-D` を付けてデバッグ情報を表示したとろ、以下のようなエラーが出ていました:

```
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@         WARNING: UNPROTECTED PRIVATE KEY FILE!          @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
Permissions 0777 for '/Volumes/data/.docker/machine/machines/default/id_rsa' are too open.
It is required that your private key files are NOT accessible by others.
This private key will be ignored.
Load key "/Volumes/data/.docker/machine/machines/default/id_rsa": bad permissions
debug2: we did not send a packet, disable method
debug1: No more authentication methods to try.
Permission denied (publickey,password,keyboard-interactive).
```

実は、 **SDカードがexFATでフォーマットされていたため、permissionが777になってしまっていた**
ことが原因でした。
exFATの場合、 `chmod` することも出来ないので、フォーマットを変更する必要があります。

Mac OS X用拡張ストレージは、 **〈OS X 拡張 (ジャーナリング)〉でフォーマット** しましょう。
また、この際に **〈大文字／小文字を区別する〉は不要** です。
OS Xのメインストレージでは区別されませんし、
Adobeなど一部の製品は大文字小文字を区別するファイルシステムをサポートしていません[^1]。
無用なトラブルを避けるためにも、注意しましょう。

[^1]: [Mac OS X ファイルシステムの確認方法と大文字と小文字を区別するファイルシステムについて](https://helpx.adobe.com/jp/x-productkb/global/cpsid_83180.html)
