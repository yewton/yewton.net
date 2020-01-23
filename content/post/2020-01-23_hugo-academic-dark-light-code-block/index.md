---
title: "Hugo Academic でダーク・ライト両モードに対応した Chroma によるシンタックスハイライト"
author: ["yewton"]
date: 2020-01-23
mylastmod: 2020-01-23T09:17:57+09:00
slug: "hugo-academic-dark-light-code-block"
categories: ["Webサイト運用"]
draft: true
image:
  caption: Background image by <a style="background-color:black;color:white;text-decoration:none;padding:4px 6px;font-family:-apple-system, BlinkMacSystemFont, &quot;San Francisco&quot;, &quot;Helvetica Neue&quot;, Helvetica, Ubuntu, Roboto, Noto, &quot;Segoe UI&quot;, Arial, sans-serif;font-size:12px;font-weight:bold;line-height:1.2;display:inline-block;border-radius:3px" href="https://unsplash.com/@markusspiske?utm_medium=referral&amp;utm_campaign=photographer-credit&amp;utm_content=creditBadge" target="_blank" rel="noopener noreferrer" title="Download free do whatever you want high-resolution photos from Markus Spiske"><span style="display:inline-block;padding:2px 3px"><svg xmlns="http://www.w3.org/2000/svg" style="height:12px;width:auto;position:relative;vertical-align:middle;top:-2px;fill:white" viewBox="0 0 32 32"><title>unsplash-logo</title><path d="M10 9V0h12v9H10zm12 5h10v18H0V14h10v9h12v-9z"></path></svg></span><span style="display:inline-block;padding:2px 3px">Markus Spiske</span></a>
---

## 前提 {#前提}

[Academic テーマ](https://sourcethemes.com/academic/) のデフォルト設定は

```emacs-lisp
(defvar modi/org-version-select 'dev
  "Variable to choose the version of Org to be loaded.
Valid values are `dev', `elpa' and `emacs'.

When set to `dev', the development version of Org built locally
is loaded.
When set to `elpa', Org is installed and loaded from Org Elpa.
When set to `emacs', the Org version shipped with Emacs is used.

The value is defaulted to `elpa' as few things in this config
need Org version to be at least 9.x.")

(defvar modi/default-lisp-directory "/your/emacs/share/dir/version/lisp/"
  "Directory containing lisp files for the Emacs installation.

This value must match the path to the lisp/ directory of your
Emacs installation.  If Emacs is installed using
--prefix=\"${PREFIX_DIR}\" this value would typically be
\"${PREFIX_DIR}/share/emacs/<VERSION>/lisp/\".")

(defvar org-dev-lisp-directory "/value/of/lispdir/in/local.mk"
  "Directory containing lisp files for dev version of Org.

This value must match the `lispdir' variable in the Org local.mk.
By default the value is \"$prefix/emacs/site-lisp/org\", where
`prefix' must match that in local.mk too.")

(defvar org-dev-info-directory "/value/of/infodir/in/local.mk"
  "Directory containing Info manual file for dev version of Org.

This value must match the `infodir' variable in the Org local.mk.")

(when (and org-dev-lisp-directory
           org-dev-info-directory)
  (with-eval-after-load 'package
    ;; If `modi/org-version-select' is *not* `emacs', remove the Emacs
    ;; version of Org from the `load-path'.
    (unless (eq modi/org-version-select 'emacs)
      ;; Remove Org that ships with Emacs from the `load-path'.
      (let ((default-org-path (expand-file-name "org" modi/default-lisp-directory)))
        (setq load-path (delete default-org-path load-path))))

    (>=e "25.0" ;`directory-files-recursively' is not available in older emacsen
        ;; If `modi/org-version-select' is *not* `elpa', remove the Elpa
        ;; version of Org from the `load-path'.
        (unless (eq modi/org-version-select 'elpa)
          (dolist (org-elpa-install-path (directory-files-recursively
                                          package-user-dir
                                          "\\`org\\(-plus-contrib\\)*-[0-9.]+\\'"
                                          :include-directories))
            (setq load-path (delete org-elpa-install-path load-path))
            ;; Also ensure that the associated path is removed from Info
            ;; search list.
            (setq Info-directory-list (delete org-elpa-install-path Info-directory-list)))))

    (let ((dev-org-path (directory-file-name org-dev-lisp-directory))
          (dev-org-info (directory-file-name org-dev-info-directory)))
      (if (eq modi/org-version-select 'dev)
          (progn
            (add-to-list 'load-path dev-org-path)
            ;; It's possible that `org-dev-info-directory' is set to an
            ;; unconventional value, in which case, it will not be
            ;; automatically added to `Info-directory-alist'. So to ensure
            ;; that the correct Org Info is used, add it to
            ;; `Info-directory-alist' manually.
            (add-to-list 'Info-directory-list dev-org-info))
        ;; If `modi/org-version-select' is *not* `dev', remove the
        ;; development version of Org from the `load-path'.
        (setq load-path (delete dev-org-path load-path))
        (with-eval-after-load 'info
          ;; Also ensure that the associated path is removed from Info search
          ;; list.
          (setq Info-directory-list (delete dev-org-info Info-directory-list)))))))
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
:END:

#+begin_src yaml :front_matter_extra t
  image:
    caption: Background image by
#+end_src
")))
```

```sh
hugo gen chromastyles --style=monokai > pbcopy
hugo gen chromastyles --style=monokailight > pbcopy
```

<https://css2sass.herokuapp.com/>

academic の pre code が勝たないようにする

```scss
code {
    background-color: #f8f8f8;
}
```

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

NabeBuiltin があるやつ
ライト
<https://xyproto.github.io/splash/docs/abap.html>
<https://xyproto.github.io/splash/docs/algol.html>
<https://xyproto.github.io/splash/docs/algol%5Fnu.html>
<https://xyproto.github.io/splash/docs/arduino.html>
<https://xyproto.github.io/splash/docs/autumn.html>
<https://xyproto.github.io/splash/docs/colorful.html>
<https://xyproto.github.io/splash/docs/emacs.html>
<https://xyproto.github.io/splash/docs/friendly.html>
<https://xyproto.github.io/splash/docs/github.html>
<https://xyproto.github.io/splash/docs/lovelace.html>
<https://xyproto.github.io/splash/docs/manni.html>
<https://xyproto.github.io/splash/docs/murphy.html>
<https://xyproto.github.io/splash/docs/pastie.html>
<https://xyproto.github.io/splash/docs/perldoc.html>
<https://xyproto.github.io/splash/docs/pygments.html>
<https://xyproto.github.io/splash/docs/rainbow%5Fdash.html>
<https://xyproto.github.io/splash/docs/solarized-light.html>
<https://xyproto.github.io/splash/docs/tango.html>
<https://xyproto.github.io/splash/docs/trac.html>
<https://xyproto.github.io/splash/docs/vim.html>
<https://xyproto.github.io/splash/docs/xcode.html>

NG
<https://xyproto.github.io/splash/docs/borland.html>
<https://xyproto.github.io/splash/docs/bw.html>
<https://xyproto.github.io/splash/docs/igor.html>
<https://xyproto.github.io/splash/docs/monokailight.html>
<https://xyproto.github.io/splash/docs/paraiso-light.html>
<https://xyproto.github.io/splash/docs/vs.html>

ダーク
<https://xyproto.github.io/splash/docs/api.html>
<https://xyproto.github.io/splash/docs/dracula.html>
<https://xyproto.github.io/splash/docs/native.html>
<https://xyproto.github.io/splash/docs/solarized-dark.html>
<https://xyproto.github.io/splash/docs/solarized-dark256.html>
<https://xyproto.github.io/splash/docs/swapoff.html>

NG

<https://xyproto.github.io/splash/docs/fruity.html>
<https://xyproto.github.io/splash/docs/monokai.html>
<https://xyproto.github.io/splash/docs/paraiso-dark.html>
<https://xyproto.github.io/splash/docs/rrt.html>
