+++
title = "Emacs でマジックコメントを反映させたいとき"
author = ["yewton"]
date = 2020-01-09
lastmod = 2020-01-09T06:29:22+09:00
slug = "find-alternate-file"
categories = ["Emacs"]
draft = true
+++

```emacs-lisp
;; -*- lexical-binding: t; -*-
```

```emacs-lisp
;; Local Variables:
;; flycheck-disabled-checkers: (emacs-lisp emacs-lisp-checkdoc)
;; no-byte-compile: t
;; End:
```

`find-alternate-file`

> Find file _FILENAME_, select its buffer, kill previous buffer.
> If the current buffer now contains an empty file that you just visited
> (presumably by mistake), use this command to visit the file you really want.

<https://superuser.com/questions/208488/how-do-i-re-open-a-file-in-emacs>
