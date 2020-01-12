+++
title = "久々の更新2"
author = ["yewton"]
date = 2019-01-05
mylastmod = 2020-01-12T06:40:11+09:00
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

<https://github.com/sourcethemes/academic-scripts> に page bunlde 化するスクリプトがある。

```sh
asdf plugin add hugo https://github.com/beardix/asdf-hugo
```

`baseurl` を設定しないと Twitter Card などが表示されないので気をつけよう

```toml
# The URL of your site.
# End your URL with a `/` trailing slash, e.g. `https://example.com/`.
baseurl = "https://www.yewton.net/"
```

`google_tag_manager` が設定されていると Google Analytics のコードが埋め込まれない
<https://spectrum.chat/academic/help/google-analytics-not-working-for-me~3e802803-663f-4b38-8d6f-bc645935da26?authed=true>
