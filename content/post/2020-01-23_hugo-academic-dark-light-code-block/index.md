---
title: "Hugo Academic でダーク・ライト両モードに対応した Chroma によるシンタックスハイライト"
author: ["yewton"]
date: 2020-01-24T08:37:00+09:00
mylastmod: 2020-01-24T08:37:59+09:00
slug: "hugo-academic-dark-light-code-block"
tags: ["emacs", "chroma", "hugo", "academic"]
categories: ["Webサイト運用"]
draft: false
image:
  caption: Background image by <a style="background-color:black;color:white;text-decoration:none;padding:4px 6px;font-family:-apple-system, BlinkMacSystemFont, &quot;San Francisco&quot;, &quot;Helvetica Neue&quot;, Helvetica, Ubuntu, Roboto, Noto, &quot;Segoe UI&quot;, Arial, sans-serif;font-size:12px;font-weight:bold;line-height:1.2;display:inline-block;border-radius:3px" href="https://unsplash.com/@markusspiske?utm_medium=referral&amp;utm_campaign=photographer-credit&amp;utm_content=creditBadge" target="_blank" rel="noopener noreferrer" title="Download free do whatever you want high-resolution photos from Markus Spiske"><span style="display:inline-block;padding:2px 3px"><svg xmlns="http://www.w3.org/2000/svg" style="height:12px;width:auto;position:relative;vertical-align:middle;top:-2px;fill:white" viewBox="0 0 32 32"><title>unsplash-logo</title><path d="M10 9V0h12v9H10zm12 5h10v18H0V14h10v9h12v-9z"></path></svg></span><span style="display:inline-block;padding:2px 3px">Markus Spiske</span></a>
---

<div class="ox-hugo-toc toc">
<div></div>

<div class="heading">&#30446;&#27425;</div>

- [前提](#前提)
- [Academic 標準のコードハイライトを無効にする](#academic-標準のコードハイライトを無効にする)
- [Hugo のコードハイライトを有効にする](#hugo-のコードハイライトを有効にする)
- [ダークモード用とライトモード用の Chroma スタイルを決める](#ダークモード用とライトモード用の-chroma-スタイルを決める)
    - [light スタイル](#light-スタイル)
    - [dark スタイル](#dark-スタイル)
- [Chroma 用の CSS を生成する](#chroma-用の-css-を生成する)
- [Academic のテーマと競合しないようにする](#academic-のテーマと競合しないようにする)

</div>
<!--endtoc-->


## 前提 {#前提}

[Academic テーマ](https://sourcethemes.com/academic/) のデフォルト設定は [ドキュメントにもある通り](https://sourcethemes.com/academic/docs/writing-markdown-latex/#code-highlighting) [highlight.js](https://highlightjs.org/) を使ったもので、
この仕組みに乗っておけば基本的には問題ありません。

ただし highlight.js には一つ問題があって… **EmacsLisp に対応していません** (Lisp には対応)。
具体的には `with-eval-after-load` のような独自のマクロや、
DocString 中のクオテーションといった EmacsLisp 方言には対応出来ません。

一方、 Hugo 標準の [Syntax Highlighting](https://gohugo.io/content-management/syntax-highlighting/) は [Chroma](https://github.com/alecthomas/chroma) を使ったもので、
こちらは **EmacsLisp に対応しています** 。

([ox-hugo](https://github.com/kaushalmodi/ox-hugo) の作者さんが過去に [Issue に挙げてくれていた](https://github.com/alecthomas/chroma/issues/43) 模様…感謝 🙏)

[CodePen](https://codepen.io/yewton/pen/RwNvdBz) と [Chroma Playground](https://swapoff.org/chroma/playground/) とでそれぞれの出力結果を比べてみるとよく分かります:

```emacs-lisp
(defvar hoge "fuga"
  "Doc String 中の `QUOTE' はどうなるかな？")

(with-eval-after-load 'foo
  (unless (eq t nil) "EmacsLisp 独自キーワードはどうなるかな？")
```

<div class="src-block-caption">
  <span class="src-block-number">ソースコード 1</span>:
  元のコード
</div>

{{< figure src="2020-01-24_05-59-12_貼り付けた画像_2020_01_24_5_58.png" caption="&#22259;1:  highlight.js w/ GitHub style" >}}

{{< figure src="2020-01-24_05-56-58_貼り付けた画像_2020_01_24_5_56.png" caption="&#22259;2:  Chroma w/ GitHub style" >}}

そこで、 **Academic を使いつつ、 Hugo 標準のハイライトの仕組みを使いたい** というのが動機となります。


## Academic 標準のコードハイライトを無効にする {#academic-標準のコードハイライトを無効にする}

[ドキュメント](https://sourcethemes.com/academic/docs/writing-markdown-latex/#highlighting-options) に書かれている通り、 `config.toml` で `params.highlight` オプションを無効にする必要があります。

[Academic Kickstart](https://sourcethemes.com/academic/) をベースにしている場合、 `params.toml` で以下のように設定します:

```toml
# Enable source code highlighting? true/false
# Documentation: https://sourcethemes.com/academic/docs/writing-markdown-latex/#highlighting-options
highlight = false
```

<div class="src-block-caption">
  <span class="src-block-number">ソースコード 2</span>:
  <code>params.toml</code>
</div>


## Hugo のコードハイライトを有効にする {#hugo-のコードハイライトを有効にする}

[Academic Kickstart](https://sourcethemes.com/academic/) をベースにしている場合、 `config.toml` で以下のように
Hugo のコードハイライトが無効にされていると思います:

```toml
[markup.highlight]
  codeFences = false  # Disable Hugo's code highlighter as it conflicts with Academic's highligher.
```

これを以下のように変更します:

```toml
[markup.highlight]
  codeFences = true
  noClasses = false
```

`noClasses = false` としているのは、 **ダーク・ライトの両方のモードに対応させるため** です。
`noClasses` が `true` の場合、スタイル指定が HTML 中に埋め込まれます。
これだとモードの変更に追従して動的にスタイルを変更するということが出来ないため、
CSS クラスだけを HTML に埋め込んでもらうようにします。


## ダークモード用とライトモード用の Chroma スタイルを決める {#ダークモード用とライトモード用の-chroma-スタイルを決める}

[Chroma Style Gallery](https://xyproto.github.io/splash/docs/) ギャラリーを参考に、
**ダークモード時に使うスタイルとライトモード時に使うスタイルをそれぞれ決めます** 。
モードの切り替わりに応じて、シンタックスハイライトのスタイル自体を変更してしまいます。

選定の際の注意事項として、 **スタイルによって細かいクラス指定に対応していない場合があります** 。

具体的には、 EmacsLisp の `defvar` などは `NameBuiltin` としてパースされ、
`.nb` というクラスが指定されるのですが、これが含まれていないスタイルがいくつかあります。

参考までに、調査した結果を以下に列挙します:


### light スタイル {#light-スタイル}


#### `NameBuiltin` が含まれるもの {#namebuiltin-が含まれるもの}

-   [abap](https://xyproto.github.io/splash/docs/abap.html)
-   [algol](https://xyproto.github.io/splash/docs/algol.html)
-   [algol\_nu](https://xyproto.github.io/splash/docs/algol%5Fnu.html)
-   [arduino](https://xyproto.github.io/splash/docs/arduino.html)
-   [autumn](https://xyproto.github.io/splash/docs/autumn.html)
-   [colorful](https://xyproto.github.io/splash/docs/colorful.html)
-   [emacs](https://xyproto.github.io/splash/docs/emacs.html)
-   [friendly](https://xyproto.github.io/splash/docs/friendly.html)
-   [github](https://xyproto.github.io/splash/docs/github.html)
-   [lovelace](https://xyproto.github.io/splash/docs/lovelace.html)
-   [manni](https://xyproto.github.io/splash/docs/manni.html)
-   [murphy](https://xyproto.github.io/splash/docs/murphy.html)
-   [pastie](https://xyproto.github.io/splash/docs/pastie.html)
-   [perldoc](https://xyproto.github.io/splash/docs/perldoc.html)
-   [pygments](https://xyproto.github.io/splash/docs/pygments.html)
-   [dash](https://xyproto.github.io/splash/docs/rainbow%5Fdash.html)
-   [light](https://xyproto.github.io/splash/docs/solarized-light.html)
-   [tango](https://xyproto.github.io/splash/docs/tango.html)
-   [trac](https://xyproto.github.io/splash/docs/trac.html)
-   [vim](https://xyproto.github.io/splash/docs/vim.html)
-   [xcode](https://xyproto.github.io/splash/docs/xcode.html)


#### `NameBuiltin` が含まれないもの {#namebuiltin-が含まれないもの}

-   [borland](https://xyproto.github.io/splash/docs/borland.html)
-   [bw](https://xyproto.github.io/splash/docs/bw.html)
-   [igor](https://xyproto.github.io/splash/docs/igor.html)
-   [monokailight](https://xyproto.github.io/splash/docs/monokailight.html)
-   [light](https://xyproto.github.io/splash/docs/paraiso-light.html)
-   [vs](https://xyproto.github.io/splash/docs/vs.html)


### dark スタイル {#dark-スタイル}


#### `NameBuiltin` が含まれるもの {#namebuiltin-が含まれるもの}

-   [api](https://xyproto.github.io/splash/docs/api.html)
-   [dracula](https://xyproto.github.io/splash/docs/dracula.html)
-   [native](https://xyproto.github.io/splash/docs/native.html)
-   [dark](https://xyproto.github.io/splash/docs/solarized-dark.html)
-   [dark256](https://xyproto.github.io/splash/docs/solarized-dark256.html)
-   [swapoff](https://xyproto.github.io/splash/docs/swapoff.html)


#### `NameBuiltin` が含まれないもの {#namebuiltin-が含まれないもの}

-   [fruity](https://xyproto.github.io/splash/docs/fruity.html)
-   [monokai](https://xyproto.github.io/splash/docs/monokai.html)
-   [dark](https://xyproto.github.io/splash/docs/paraiso-dark.html)
-   [rrt](https://xyproto.github.io/splash/docs/rrt.html)


## Chroma 用の CSS を生成する {#chroma-用の-css-を生成する}

スタイルを決めたら以下のように CSS を生成します。
生成された CSS は標準出力に吐き出されるため、適当なファイルにリダイレクトするか、 `pbcopy` 等にパイプしてクリップボードに格納しましょう:

```sh
hugo gen chromastyles --style=pygments
hugo gen chromastyles --style=native
```

この後多少手を加える必要があるので、やりやすいように [CSS 2 SASS/SCSS CONVERTER](https://css2sass.herokuapp.com/) 等で SCSS に変換しておくとよいです。


## Academic のテーマと競合しないようにする {#academic-のテーマと競合しないようにする}

生成した CSS(SCSS) は [Academic のドキュメント](https://sourcethemes.com/academic/docs/customization/#customize-style-css) に従い `custom.css` に追加します。

ただし、Academic には標準のハイライトのためのスタイル指定があるため、
Chroma が生成したスタイル指定をそのまま組込むと若干コンフリクトします。

そこで、多少手を加えてやります。

まずライトモード時のスタイルについては、
`background-color` や `color` が指定されていない場合があるため( `pygments` 等)、
[Chroma Playground](https://swapoff.org/chroma/playground/) の出力結果を参考にしてスタイル指定を追加します。

また、 Academic 組込みの `code` へのスタイル指定より優先度が高くなるように、
セレクタを `pre.chroma, .chroma code` とします。

最終的に以下のようになります:

```scss
pre.chroma, .chroma code {

    background-color: #f5f5f5;
    color: #4a4a4a;
```

次にダークモード時のスタイルについては、
基本的に `background-color` や `color` も指定されていると思うので、
セレクタのみ `.dark pre.chroma, .dark .chroma code` としてあげます。

以上で、 **Hugo Academic を使いつつ、 Chroma でシンタックスハイライト** が実現出来ました。
