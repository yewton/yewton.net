+++
title = "org-download と ox-hugo を組み合わせて記事への画像挿入を快適にする"
author = ["yewton"]
date = 2020-01-09
mylastmod = 2020-01-13T16:02:47+09:00
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

(use-package org
  :config
  (setq org-use-property-inheritance '("DIR")))

(require 's)
(defun ytn-new-post-template ()
  (let* ((slug (read-string "Slug: "))
         (now (current-time))
         (export-date (format-time-string "%Y-%m-%d" now))
         (id (format "%s_%s" export-date slug)))
    (s-lex-format "\
* DRAFT %?
:PROPERTIES:
:EXPORT_FILE_NAME:   index
:EXPORT_DATE:        ${export-date}
:DIR:                images/${id}
:EXPORT_HUGO_BUNDLE: ${id}
:EXPORT_HUGO_SLUG:   ${slug}
:END:")))

(setq org-capture-templates '(("n" "サイト用")
                              ("np" "New post" entry (file+headline "~/Projects/yewton.net/content-org/all-posts.org" "Inbox")
                               (function ytn-new-post-template) :empty-lines 1 :prepend t)
                              ))
```
