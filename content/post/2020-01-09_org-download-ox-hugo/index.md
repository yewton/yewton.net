+++
title = "org-download と ox-hugo を組み合わせて記事への画像挿入を快適にする"
author = ["yewton"]
date = 2020-01-09
mylastmod = 2020-01-12T06:40:13+09:00
slug = "org-download-ox-hugo"
tags = ["emacs", "ox hugo", "org download", "org attach"]
categories = ["Emacs"]
draft = true
+++

```emacs-lisp
(use-package org-download
  :after org
  :functions (org-link-escape)
  :config
  (setq org-download-method 'attach)
  (setq org-download-link-format-function
        (lambda (filename)
          (format org-download-link-format
                  (org-link-escape
                   (funcall org-download-abbreviate-filename-function filename)))))
  (setq org-download-annotate-function
        (lambda (link)
          (format "#+DOWNLOADED: %s @ %s\n#+ATTR_ORG: :width 500\n"
                  (if (equal link org-download-screenshot-file)
                      "screenshot"
                    link)
                  (format-time-string "%Y-%m-%d %H:%M:%S"))))
  (add-hook 'dired-mode-hook 'org-download-enable))
```
