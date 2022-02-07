---
title: "consult-ripgrep と migemo を組み合わせて ripgrep でもローマ字日本語検索"
author: ["yewton"]
date: 2022-02-07T23:29:00+09:00
mylastmod: 2022-02-07T23:29:06+09:00
slug: "consult-ripgrep-migemo"
tags: ["migemo", "ripgrep", "consult", "emacs"]
categories: ["Emacs"]
draft: false
---

[consult-ripgrep と migemo を組み合わせて ripgrep でもローマ字日本語検索](/2022/02/07/consult-ripgrep-migemo/) の派生。


## TL;DR {#tl-dr}

```emacs-lisp
(defun consult--migemo-regexp-compiler (input type)
  (setq input (mapcar #'migemo-get-pattern (consult--split-escaped input)))
  (cons (mapcar (lambda (x) (consult--convert-regexp x type)) input)
        (when-let (regexps (seq-filter #'consult--valid-regexp-p input))
          (lambda (str)
            (consult--highlight-regexps regexps str)))))
(setq migemo-options '("--quiet" "--nonewline" "--emacs"))
(setq consult--regexp-compiler #'consult--migemo-regexp-compiler)
```


## `cosult-ripgrep` の仕組み {#cosult-ripgrep-の仕組み}

`consult-ripgrep` で指定する文字列は、 [こちら](https://github.com/minad/consult/blob/1a6ed29e92f00266daff4ff5f62602f53ef7d158/consult.el#L4311-L4318) にあるように構造化されている。

特に `#first#second` といった指定の場合に、
デフォルトでは区切り文字の前半(例の `first` の部分)は基本的にそのまま `rg` に渡され、
後半(例の `second` の部分)は `rg` の結果について更に Emacs 上で絞り込む、といった挙動になる
( 基本的に、と書いたのは Emacs の正規表現から `rg` に渡せる正規表現に変換してくれるという
`counsel-rg` と同じ似たような機構が備わっているから。[この辺](https://github.com/minad/consult/blob/1a6ed29e92f00266daff4ff5f62602f53ef7d158/consult.el#L571-L590) )。

後者については [`completion-styles`](https://www.gnu.org/software/emacs/manual/html%5Fnode/emacs/Completion-Styles.html) の指定通りの補完スタイルが適用される為、
[consultをmigemoizeしたい (未完→だいたいできた) - ワタタツの日記!(2021-06-15)](https://nyoho.jp/diary/?date=20210615) の記事にあるように
`completion-styles` を設定していれば migemo が使用されるが、
前者については `consult-ripgrep` 独自の仕組みなので `completion-styles` には影響されない。


## `consult--regexp-compiler` {#consult-regexp-compiler}

[consult-grep の DOCSTRING](https://github.com/minad/consult/blob/1a6ed29e92f00266daff4ff5f62602f53ef7d158/consult.el#L4306-L4309) に以下の記載がある:

> In order to disable transformations of the grep input, adjust \`consult--regexp-compiler' accordingly.

[デフォルトの実装](https://github.com/minad/consult/blob/1a6ed29e92f00266daff4ff5f62602f53ef7d158/consult.el#L592-L602) はこんな感じ:

```emacs-lisp
(defun consult--default-regexp-compiler (input type)
  "Compile the INPUT string to a list of regular expressions.
The function should return a pair, the list of regular expressions and a
highlight function. The highlight function should take a single argument, the
string to highlight given the INPUT. TYPE is the desired type of regular
expression, which can be `basic', `extended', `emacs' or `pcre'."
  (setq input (consult--split-escaped input))
  (cons (mapcar (lambda (x) (consult--convert-regexp x type)) input)
        (when-let (regexps (seq-filter #'consult--valid-regexp-p input))
          (lambda (str)
            (consult--highlight-regexps regexps str)))))
```

入力文字列から最終的に `rg` に渡される文字列を組み立てているここに、介入余地がある。

ということで、冒頭のような設定を行い、 migemo 対応版の `consult--regexp-compiler` を指定してやればよい。

{{% callout warning %}}
`consult--regexp-compiler` はその命名からも分かる通り、あくまで内部変数のようである。
その為、今後の更新によって使えなくなる可能性も十分にある。
{{% /callout %}}
