+++
title = "auto-commit-and-push のこと書きたい"
author = ["yewton"]
date = 2020-01-09
lastmod = 2020-01-09T09:00:53+09:00
slug = "auto-commit-and-push"
categories = ["Emacs"]
draft = true
+++

```emacs-lisp
((nil . ((eval git-auto-commit-mode 1)
         (gac-automatically-push-p . t))))
```

<div class="src-block-caption">
  <span class="src-block-number">ソースコード 1</span>:
  .dir-locals.el
</div>

`git-auto-commit-mode` は autoload されるので、 autoload が適切に扱われていれば `require` は不要。
