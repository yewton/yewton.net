+++
title = "org-mode に Web のリンクを貼りたい"
author = ["yewton"]
date = 2020-01-09
lastmod = 2020-01-09T13:58:30+09:00
slug = "org-mode-web-link"
categories = ["未分類"]
draft = true
+++

<https://orgmode.org/Changes.html>

> Version 9.3
> Incompatible changes
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

```text
%text%\n%url%
```
