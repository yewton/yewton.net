---
title: "Ivy (Swiper) で雑に migemo を使う"
author: ["yewton"]
date: 2020-05-21T04:50:00+09:00
mylastmod: 2021-02-27T22:00:00+09:00
slug: "migemo-ivy"
tags: ["ivy", "migemo", "swiper", "emacs"]
categories: ["Emacs"]
draft: false
---

{{< lastmod >}}

{{< callout note >}}
[counsel-rg と migemo を組み合わせて ripgrep でもローマ字検索](/2020/05/24/counsel-rg-migemo/) も併せて参照。
{{</ callout >}}


## TL;DR {#tl-dr}

```emacs-lisp
(reqiure 'dash)
(require 's)

(defun ytn-ivy-migemo-re-builder (str)
  (let* ((sep " \\|\\^\\|\\.\\|\\*")
         (splitted (--map (s-join "" it)
                          (--partition-by (s-matches-p " \\|\\^\\|\\.\\|\\*" it)
                                          (s-split "" str t)))))
    (s-join "" (--map (cond ((s-equals? it " ") ".*?")
                            ((s-matches? sep it) it)
                            (t (migemo-get-pattern it)))
                      splitted))))

(setq ivy-re-builders-alist '((t . ivy--regex-plus)
                              (swiper . ytn-ivy-migemo-re-builder)))
```


## 背景 {#背景}

Swiper で migemo を使おうと思ってググると [avy-migemo](https://github.com/momomo5717/avy-migemo) での設定例しか出てこない。

個人的には avy を使っていない(使えていない)のと、
Swiper を使いたいのに avy が絡んでくるっていうのがよく分からなかったので利用するのが憚られた。

そこで、 ivy の仕組みをちゃんと理解してやってみようと思った。


## migemo の仕組み {#migemo-の仕組み}

migemo 自体の仕組みは超単純で、 [Migemo: ローマ字のまま日本語をインクリメンタル検索](http://0xcc.net/migemo/) で解説されている通り:

> 利用者が 1文字入力するたびに、ローマ字列から正規表現を生成して、それで検索するという力技な方法です。

この正規表現を取得するには `migemo-get-pattern` を使う:

```text
ELISP> (migemo-get-pattern "ebihurai")
"\\(ｴ\\s-*ﾋ\\s-*ﾞ\\s-*ﾌ\\s-*ﾗ\\s-*ｲ\\|エ\\s-*ビ\\s-*フ\\s-*ラ\\s-*イ\\|海\\s-*老\\s-*フ\\s-*ラ\\s-*イ\\|え\\s-*び\\s-*ふ\\s-*ら\\s-*い\\|ｅ\\s-*ｂ\\s-*ｉ\\s-*ｈ\\s-*ｕ\\s-*ｒ\\s-*ａ\\s-*ｉ\\|e\\s-*b\\s-*i\\s-*h\\s-*u\\s-*r\\s-*a\\s-*i\\)"
```


## ivy の仕組み {#ivy-の仕組み}

`ivy-read` という関数が、入力からいい感じの正規表現を使って検索してくれるフロントエンド。

そこで使われる正規表現は、 `ivy-alist-setting` を通して `ivy-re-builders-alist` から取得される。

Swiper の場合はこんな感じ:

```emacs-lisp
(ivy-read
                 "Swiper: "
                 candidates
                 :initial-input initial-input
                 :keymap swiper-map
                 :preselect preselect
                 :require-match t
                 :update-fn #'swiper--update-input-ivy
                 :unwind #'swiper--cleanup
                 :action #'swiper--action
                 :re-builder #'swiper--re-builder
                 :history 'swiper-history
                 :caller 'swiper)
```

migemo を使いたいのは今のところ Swiper だけなので、 `'swiper` をキーにして独自関数を設定すればいい。

```emacs-lisp
(setq ivy-re-builders-alist '((t . ivy--regex-plus)
                              (swiper . ytn-ivy-migemo-re-builder)))
```


## migemo を使って検索する独自関数 {#migemo-を使って検索する独自関数}

Swiper はスペースで区切って雑に絞り込んでいけるというのが最大の(個人的な)メリットなので、最低限それを実現出来ればよかった。

実は `!` で exclude を実現出来たり色々他にも気の効いた機能があるみたいだけれど、自分は使っていない(そこまで使いこなせてない)。

なので、空白や正規表現っぽい入力は as-is にして、それ以外の入力を migemo に食わせてやり、最後に空白を `.*?` に変換して結合した文字列を最終的な正規表現とする。

というのを実現しようとするとこんな感じになる:

```emacs-lisp
(defun ytn-ivy-migemo-re-builder (str)
  (let* ((sep " \\|\\^\\|\\.\\|\\*")
         (splitted (--map (s-join "" it)
                          (--partition-by (s-matches-p " \\|\\^\\|\\.\\|\\*" it)
                                          (s-split "" str t)))))
    (s-join "" (--map (cond ((s-equals? it " ") ".*?")
                            ((s-matches? sep it) it)
                            (t (migemo-get-pattern it)))
                      splitted))))
```
