+++
title = "org-mode に Chrome で開いてるページへのリンクを貼りたい"
author = ["yewton"]
date = 2020-01-10
lastmod = 2020-01-10T09:02:59+09:00
slug = "org-mode-web-link"
tags = ["emacs", "org mode"]
categories = ["Emacs"]
draft = false
+++

`org-mode` のリンクマークアップはちょっと特殊で、エスケープの仕様も独特です。

また、  [2019年の12月にリリースされたバージョン 9.3 で後方互換性の無い仕様変更が入る](https://code.orgmode.org/bzg/org-mode/src/release%5F9.3/etc/ORG-NEWS) ということも有りました。

> **Version 9.3**
>
> **Incompatible changes**
>
> Change bracket link escaping syntax
> Org used to percent-encode sensitive characters in the URI part of the bracket links.
>
> Now, escaping mechanism uses the usual backslash character, according to the following rules, applied in order:

```emacs-lisp
(defun org-link-unescape (link)
  "Remove escaping backslash characters from string LINK."
  (replace-regexp-in-string
   (rx (group (one-or-more "\\")) (or string-end (any "[]")))
   (lambda (_)
     (concat (make-string (/ (- (match-end 1) (match-beginning 1)) 2) ?\\)))
   link nil t 1))
```

<div class="src-block-caption">
  <span class="src-block-number">ソースコード 1</span>:
  (参考) <code>org-mode</code> 9.3 でのエスケープ実装
</div>

こういう背景もあり、 `org-mode` の外で工夫して `org-mode` 形式のリンクを生成するよりは、
 `org-mode` 自体に任せてしまうのが安心です。

[org-cliplink](https://github.com/rexim/org-cliplink) というパッケージもありますが、これだとログインが必要なページへのリンクは
(Basic 認証を設定していなければ)生成出来ません。

もっと手軽に、 Chrome で見ているページへのリンクを挿入する手段として、
[CreateLink](https://github.com/ku/CreateLink) という Chrome 拡張機能を使う方法があります。

以下のように改行区切りでコピーするような設定を追加します:

```text
%text%%newline%%url%
```

CreateLink の `%text%` は改行をスペースに変換する仕様になっているので、区切り文字として改行を利用するのは安全なハズです。

そして、以下のような独自関数を定義します:

```emacs-lisp
(require 's)

(defun ytn-org-insert-weblink ()
  (interactive)
  (let* ((pair (s-split "\n" (with-temp-buffer (clipboard-yank) (buffer-string))))
         (desc (first pair))
         (link (second pair)))
    (insert (org-make-link-string link desc))))
```

単純に改行で区切って `org-make-link-string` に渡すだけです。

これを、個人的には <kbd>M-L</kbd> にアサインしています([use-package](https://github.com/jwiegley/use-package) の `bind-key` を利用しています):

```emacs-lisp
(bind-key "M-L" #'ytn-org-insert-weblink org-mode-map)
```

以上、ちょっとした小ネタでした。
