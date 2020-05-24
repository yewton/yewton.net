---
title: "counsel-rg と migemo を組み合わせて ripgrep でもローマ字検索"
author: ["yewton"]
date: 2020-05-25T06:51:00+09:00
mylastmod: 2020-05-25T06:51:43+09:00
slug: "counsel-rg-migemo"
tags: ["migemo", "ripgrep", "ivy", "emacs"]
categories: ["Emacs"]
draft: false
---

[Ivy (Swiper) で雑に migemo を使う](/2020/05/21/migemo-ivy/) の続編。併せて参照のコト。


## TL;DR {#tl-dr}

```emacs-lisp
(setq migemo-options '("--quiet" "--nonewline" "--emacs"))
(setq ivy-re-builders-alist '((t . ivy--regex-plus)
                              (counsel-rg . ytn-ivy-migemo-re-builder)
                              (swiper . ytn-ivy-migemo-re-builder)))
```


## counsel-rg の仕組み {#counsel-rg-の仕組み}

Emacs で使える正規表現と rg に渡せる正規表現は異なるのに、
そもそも一体どうやって `counsel-rg` が実現されているかを確認した。

すると、 `counsel--elisp-to-pcre` という関数で、
Emacs 式正規表現を [PCRE (Perl Compatible Regular Expressions)](https://www.pcre.org/) にベストエフォートで変換してくれている。


## migemo(cmigemo) が出力する正規表現と、 PCRE 非互換の問題 {#migemo--cmigemo--が出力する正規表現と-pcre-非互換の問題}

デフォルトでは以下のような正規表現が出力される:

```text
$ cmigemo -d '/usr/local/Cellar/cmigemo/20110227/share/migemo/utf-8/migemo-dict' --quiet --emacs --word wagahai
\(ﾜ\s-*ｶ\s-*ﾞ\s-*ﾊ\s-*ｲ\|ワ\s-*ガ\s-*ハ\s-*イ\|吾\s-*\(輩\|が\s-*輩\)\|我\s-*\(輩\|が\s-*輩\)\|わ\s-*が\s-*は\s-*い\|ｗ\s-*ａ\s-*ｇ\s-*ａ\s-*ｈ\s-*ａ\s-*ｉ\|w\s-*a\s-*g\s-*a\s-*h\s-*a\s-*i\)
```

これは、空白や改行を挟んでいてもマッチするようにするため。

この `\s-` というのは Emacs 独自のもので、シンタックステーブルを利用したマッチの仕組み:

> ```text
> ‘\sCODE’
>      matches any character whose syntax is CODE.  Here CODE is a
>      character that represents a syntax code: thus, ‘w’ for word
>      constituent, ‘-’ for whitespace, ‘(’ for open parenthesis, etc.  To
>      represent whitespace syntax, use either ‘-’ or a space character.
>      *Note Syntax Class Table::, for a list of syntax codes and the
>      characters that stand for them.
> ```

`\s-` は構文上ホワイトスペース扱いされる文字にマッチする。

これは当然、他の正規表現エンジンでは使えない。

PCRE では `\s` がホワイトスペースにマッチするので(参考: [pcrepattern specification](https://www.pcre.org/original/doc/html/pcrepattern.html#genericchartypes))、
`\s-` を `\s` で置換する、という手もあるけれど、
そもそも ripgrep や Swiper を使うときに、ホワイトスペースが間に挟まる場合を考慮したくなるということは少ないと思うので、
複数行を考慮させないようにするという方向で調整したい。


## migemo(cmigemo) で複数行に渡る文字列を考慮しないようにする {#migemo--cmigemo--で複数行に渡る文字列を考慮しないようにする}

そのものズバリなオプションが用意されている:

> ````text
> -n --nonewline        Don't use newline match.
> ````

このオプションを指定すると、出力される正規表現は以下のようになる:

`````text
cmigemo -d '/usr/local/Cellar/cmigemo/20110227/share/migemo/utf-8/migemo-dict' --quiet --emasc --nonewline --word wagahai
(ﾜｶﾞﾊｲ|ワガハイ|吾(輩|が輩)|我(輩|が輩)|わがはい|ｗａｇａｈａｉ|wagahai)
`````

あとは、このオプションを `migemo-options` に指定すればよい:

`````emacs-lisp
(setq migemo-options '("--quiet" "--nonewline" "--emacs"))
`````

なお、すでに `migemo-init` している場合は反映されないので、その場合は一度 `migemo-kill` する必要がある。
