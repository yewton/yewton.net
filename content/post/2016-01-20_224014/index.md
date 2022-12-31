+++

categories = ["Emacs"]
date = "2016-01-20T22:40:16+09:00"
slug = "markdown-mode-skk-kakutei"
tags = ["emacs", "skk-mode", "markdown-mode"]
title = "markdown-modeでSKKの変換確定するためにRETするとカーソルが行頭に飛ぶ問題の回避策"

+++

markdown-mode 2.1 で SKK 15.2 使ってると、確定しようと思って `<return>` すると、
確定後にカーソルが行頭に飛んでしまうという問題に遭遇した。
`C-j` で確定する場合は起こらない。

## TL;DR ##

以下を `init.el` 相当のファイルに書けば回避出来る。

```lisp
(defun my--markdown-entery-key-ad (this-func &rest args)
  "markdown-modeでskk-henkan-mode中にエンターすると行頭にカーソルが飛んでしまう問題の対応"
  (if skk-henkan-mode (skk-kakutei)
    (apply this-func args)))
(advice-add #'markdown-enter-key :around #'my--markdown-entery-key-ad)
```

## 原因解明に至るまで ##

`<return>` = `<C-m>` では起こり、`C-j` では起こらないので、まずはキーバインドを確認したところ、
前者は `markdown-enter-key` という関数が割り当てられていた。

実装は至ってシンプルだった:

```lisp
(defun markdown-enter-key ()
  "Handle RET according to to the value of `markdown-indent-on-enter'."
  (interactive)
  (newline)
  (when markdown-indent-on-enter
    (markdown-indent-line)))
```

次に `trace-function` で `markdown-enter-key` と `skk-kakutei` をトレースしたところ、
以下のような出力が得られた:

```
1 -> (markdown-enter-key)
| 2 -> (skk-kakutei)
| 2 <- skk-kakutei: nil
1 <- markdown-enter-key: nil
```

`markdown-enter-key` が発動して `(newline)` が評価された結果、
`skk-kakutei` が発動して変換確定、その後に `markdown-indent-line` が呼ばれているのがどうも悪いらしい。

## 回避方法 ##

これは **`markdown-enter-key` が呼ばれた時に `skk-henkan-mode` 中だったら、**
**本来の挙動ではなく `skk-kakutei` だけを行なうようにする** ことで回避出来そうだ。

こういう部分的な挙動の修正には advice を使うといい。
というわけで前述の挙動をそのまま定義すると、冒頭のようなコードになる。
