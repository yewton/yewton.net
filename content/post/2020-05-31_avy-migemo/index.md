---
title: "avy で migemo る (avy-migemo を使わずに)"
author: ["yewton"]
date: 2020-05-31T05:31:00+09:00
mylastmod: 2020-05-31T05:33:32+09:00
slug: "avy-migemo"
tags: ["migemo", "avy", "emacs"]
categories: ["Emacs"]
draft: false
---

## TL;DR {#tl-dr}

```emacs-lisp
(require 'avy)
(require 'migemo)

(defun avy-goto-migemo-timer (&optional arg)
  (interactive "P")
  (let ((avy-all-windows (if arg
                             (not avy-all-windows)
                           avy-all-windows)))
    (avy-with avy-goto-migemo-timer
      (setq avy--old-cands (avy--read-candidates #'migemo-get-pattern))
      (avy-process avy--old-cands))))
(add-to-list 'avy-styles-alist '(avy-goto-migemo-timer . pre))
(global-set-key (kbd "C-M-'") 'avy-goto-migemo-timer)
```


## 背景 {#背景}

[以前の記事](/2020/05/21/migemo-ivy/) を書いた時点では [avy](https://github.com/abo-abo/avy) を使っていなかったが、改めて調べると超絶便利だったので使うことにした。

そして当然の帰結として、 [Migemo](http://0xcc.net/migemo/) に対応させたくなった。

[avy-migemo](https://github.com/momomo5717/avy-migemo) の存在は以前から知っていたけれど、色々な機能が提供されていてよく分からなかったのと、
自分がやりたいユースケースだけ考えると複雑な仕組みは要らなさそうに思えたので、
改めて `avy` の仕組みを調べて設定しようと考えた。


## やりたき事 {#やりたき事}

`avy` には色々な使い方がある。

1文字だけで候補を表示したり、2文字まで入力出来るようにしたり。
一定時間以内に入力された文字を候補とする、というような機能もある。

`migemo` との組み合わせを考えると、固定の文字数よりは任意の文字数入力出来るのが望ましい。

上記を踏まえると、
**一定時間以内に入力された文字を migemo に食わせて得られた正規表現にマッチする物を候補とする**
ような動きが実現出来るとよさそうだった。


## `avy-goto-char-timer` {#avy-goto-char-timer}

`avy` に組込まれている「一定時間以内に入力された文字を候補とする」関数定義を見てみる:

```emacs-lisp
(defun avy-goto-char-timer (&optional arg)
  "Read one or many consecutive chars and jump to the first one.
The window scope is determined by `avy-all-windows' (ARG negates it)."
  (interactive "P")
  (let ((avy-all-windows (if arg
                             (not avy-all-windows)
                           avy-all-windows)))
    (avy-with avy-goto-char-timer
      (setq avy--old-cands (avy--read-candidates))
      (avy-process avy--old-cands))))
```

案外短い。

キモは `avy--read-candidates` で、こいつはオプショナルで文字列から正規表現を作る関数を指定出来る。
ここで `migemo-get-pattern` を食わせてやればいい。


## migemo を使って候補を表示する独自関数 {#migemo-を使って候補を表示する独自関数}

というわけで冒頭の独自関数を定義する:

```emacs-lisp
(defun avy-goto-migemo-timer (&optional arg)
  (interactive "P")
  (let ((avy-all-windows (if arg
                             (not avy-all-windows)
                           avy-all-windows)))
    (avy-with avy-goto-migemo-timer
      (setq avy--old-cands (avy--read-candidates #'migemo-get-pattern))
      (avy-process avy--old-cands))))
```

`avy-goto-char-timer` をほんの一部書き換えただけ。

デフォルトだと候補のキーは文字に重ねて表示されるが、
migemo で引っ掛けたいような文字は入力した文字とは異なる場合が多い。
そのため重ねずに前方に表示するようにしたいため、明示的にスタイルを設定しておく:

```emacs-lisp
(add-to-list 'avy-styles-alist '(avy-goto-migemo-timer . pre))
```
