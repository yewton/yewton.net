+++
title = "久々の更新2"
author = ["yewton"]
date = 2019-01-05
slug = "random"
tags = ["emacs", "hugo", "ox-hugo"]
categories = ["雑記"]
draft = true
+++

org-mode ではタグにハイフンを付けられない。

`ox_hugo` <kbd>hoge</kbd>

conf-toml がある <https://github.com/dryman/toml-mode.el/issues/14>

```emacs-lisp
(use-package ox-hugo
  :after ox
  :init
  (defalias 'toml-mode 'conf-toml-mode)
  :config
  (setq org-hugo-use-code-for-kbd t))
```

```sh
hugo gen chromastyles --style=monokai > pbcopy
hugo gen chromastyles --style=monokailight > pbcopy
```

<https://css2sass.herokuapp.com/>

`.np` とか、light にしか無いやつを消した(暫定)

<https://gohugo.io/getting-started/configuration-markup#highlight>

```toml
[markup.highlight]
  codeFences = true
  guessSyntax = false
  hl_Lines = ""
  lineNoStart = 1
  lineNos = false
  lineNumbersInTable = true
  noClasses = false
  style = "emacs"
  tabWidth = 4
```
