+++
banner = "banners/emacs.jpg"
categories = ["雑記"]
date = "2016-05-04T19:45:26+09:00"
slug = "helm-persp"
tags = ["emacs", "helm", "persp-mode"]
title = "Spacemacs から helm と persp-mode の設定をパクろうと思ったけど難しかった"

+++

[この辺](https://github.com/syl20bnr/spacemacs/blob/master/layers/%2Bwindow-management/spacemacs-layouts/funcs.el) を参考にせよ、
と persp-mode 公式に書いてあったので、パクれそうかやってみた。

結論としては、出来なかった。
なので、 併せて記載されていた [こっち](https://gist.github.com/Bad-ptr/304ada85c9ba15013303) の設定を使う状態のまま。

Spacemacs の設定は、 Spacemacs 独自のステートの概念(Vimmer が喜ぶやつ)と密接に関連しているようで、
素の Emacs に組込むのは骨が折れそうだった。

そもそも何で Spacemacs の設定をパクろうとしていたかというと、
後者の設定だと、 `C-x b` したときに前のバッファが選択されずに
現在のバッファが選択された状態になるのが違和感があったから、だった。
素の Emacs では単に `C-x b RET` としたときは、前のバッファに戻るという挙動になる。
もしかしたら Spacemacs では元の挙動を再現した実装になっているのかもしれない、
と思って見てみたが、前述の通り失敗に終わった。

persp-mode では、直前のバッファが現在のパースペクティブに含まれているとは限らないから、仕方ないのかな…。
