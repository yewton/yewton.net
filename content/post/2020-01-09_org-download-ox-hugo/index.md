+++
title = "org-download と ox-hugo を組み合わせて記事への画像挿入を快適にする"
author = ["yewton"]
date = 2020-01-09
lastmod = 2020-01-09T06:23:33+09:00
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

```org
#+DOWNLOADED: file:/Users/yewton/Downloads/drive-download-20200107T213252Z-001/Screenshot_2020-01-08-01-02-47-757.jpeg @ 2020-01-09 06:15:06
#+ATTR_ORG: :width 500
[[file:images/2020-01-08_lonovo-tab-m8/2020-01-09_06-15-06_Screenshot_2020-01-08-01-02-47-757.jpeg]]
```
