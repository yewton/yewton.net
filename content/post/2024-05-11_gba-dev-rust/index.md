---
title: "Rust で GBA 開発のとっかかり"
author: ["yewton"]
date: 2024-05-12T00:02:00+09:00
mylastmod: 2024-05-12T00:02:49+09:00
slug: "gba-dev-rust"
tags: ["gba", "rust", "agb"]
categories: ["技術系"]
draft: false
---

<div class="ox-hugo-toc toc">
<div></div>

<div class="heading">&#30446;&#27425;</div>

- [背景](#背景)
- [GBA 開発について](#gba-開発について)
- [agb での開発](#agb-での開発)
- [成果物](#成果物)
- [終わりに](#終わりに)

</div>
<!--endtoc-->


## 背景 {#背景}

新しい言語を学ぶキッカケとして、 2015 年頃にほんの少しだけ触っていた GameBoy Advance の開発を
[Rust](https://www.rust-lang.org/) で出来ないかと検討していた。

当時  [libgba](https://github.com/devkitPro/libgba) を利用してつくったものは以下。これらを Rust で書き直すことが目標。


### jumpingdroid {#jumpingdroid}

{{< figure src="2024-05-11_20-44-18_jumpingdroid_mb-1.png" caption="&#22259;1:  ドロイド君が跳ねたり銀の林檎に乗ったりする" >}}

単純な背景と、複数スプライトの配置を試してみたもの。

キャラクターを動かして何かに乗せる ( ように見せる ) 、というだけのことをやってみた。


### jumpingdroid2 {#jumpingdroid2}

{{< figure src="2024-05-11_20-57-33_jumpingdroid2_mb-1.png" caption="&#22259;2:  ドロイド君が右へ右へと進んでいく。ボタン押下で色も変わる。" >}}

1 画面に収まらないステージを横にスクロールしながら進んでいくだけのもの。

画面外で次々に進行方向の背景を描画していくことで、キャラクターが進んでいるように見せる、ということをやってみた。


## GBA 開発について {#gba-開発について}

GBA の開発は普通のコンピュータ上で行なうようなものとは大分様相が異なる。

OS なんてものはなく、「見渡す限りビットだらけ」である。

> There is no operating system, no messing with drivers and hardware incompatibilities; it's bits as far as the eye can see.
>
> --  [_Tonc: GBA Hardware_](https://www.coranac.com/tonc/text/hardware.htm)

メモリの番地 `0x06000000` からが背景画像のデータ領域で、 `0x06010000` からはキャラクターのスプライト用のデータ、
個々のスプライトは `0x07000000` から始まる領域で管理される、
というように、 **ある特定のメモリ領域のビットを操作したらそれが画面に反映される** というのが基本的な仕組み。

{{< figure src="2024-05-11_21-43-09_screenshot.png" caption="&#22259;3:  ドロイド君の左下部分の画像が `0x06010040` にある様子" >}}

jumpingdroid では BG (背景画像)とスプライトの基本的な機能を利用している。


### BG {#bg}

GBA では最大 4 枚の背景画像を利用でき、256×256 ピクセルや 256×512 ピクセルなどのサイズに設定出来る。

{{< figure src="2024-05-11_22-06-15_screenshot.png" caption="&#22259;4:  256×256 ピクセルの BG" >}}

GBA の画面は 240×160 ピクセルなので、上記の内実際に表示されるのは中央部分のみ。
スクロール表示の実現の為には、この非表示部分と表示オフセットを上手く利用することが必要になる。


### スプライト {#スプライト}

スプライトは小さな画像オブジェクトで、個別に移動させたり反転させたりといった操作が可能。
GBA では 8×8 から 64×64 ピクセルで、最大 128 個のスプライトを扱える。

{{< figure src="2024-05-11_22-15-29_screenshot.png" caption="&#22259;5:  `0x07000000` に格納されているドロイド君のスプライトデータ。参照先タイルは `0x06010180` にある。" >}}


### 高速紙芝居 {#高速紙芝居}

無限ループの中で **各ループ内で画面がどういう状態になるかをひたすら更新していく** 、というのが基本的なイメージ。

各オブジェクトの状態とプレイヤーによるボタン入力の状態に応じて高速で画面を書き換えていくと動いているように見えるという、単純な仕組み。

単純ゆえに、例えばただキャラクターを動かすといったことでも、全てゼロから定義する必要がある。

移動や拡大・回転といった基本的な機能は提供されているが、それらをどう利用するかは完全に開発者次第。


## agb での開発 {#agb-での開発}

今回利用する [agb](https://agbrs.dev/) というライブラリでは、 GBA でのゲーム開発に必要な機能をかなり抽象化して扱い易くしてくれている。

スプライトは [Aseprite](https://www.aseprite.org/) というピクセルアート・アニメーション作成ツールのフォーマットをそのまま利用出来る
( Aseprite は購入するか、自身でビルドする必要がある )。

{{< figure src="2024-05-11_23-00-28_screenshot.png" caption="&#22259;6:  Aseprite の編集画面" >}}

```rust
const GRAPHICS: &Graphics = agb::include_aseprite!("gfx/sprites.aseprite");
const TAG_MAP: &TagMap = GRAPHICS.tags();

const IDLE: &Tag = TAG_MAP.get("Idle");
const WALKING: &Tag = TAG_MAP.get("Walking");
const JUMPING: &Tag = TAG_MAP.get("Jumping");
```

<div class="src-block-caption">
  <span class="src-block-number">ソースコード 1</span>:
  コードからのスプライト用データ利用イメージ
</div>

BG データは  png 等もそのまま扱える。

```rust
agb::include_background_gfx!(tiles,
    "ff00ff", // 透過色
    bg => "gfx/bg.png");
```

<div class="src-block-caption">
  <span class="src-block-number">ソースコード 2</span>:
  コードからのBGデータ利用イメージ
</div>

これらはコンパイル時に GBA での利用に適した形に変換されているらしい。

プログラム全体の構造をざっくり書くと以下のようになる:

```rust
#[agb::entry]
fn main(mut gba: agb::Gba) -> ! {
    // 初期化
    let object = gba.display.object.get_managed();
    let mut droid_object = object.object_sprite(IDLE.sprite(0));
    let mut bg0 = gfx.background(
        Priority::P0,
        RegularBackgroundSize::Background32x32,
        TileFormat::FourBpp
    );
    // ...
    loop {
        // フレーム毎に色々計算
        // ...
        // BG のオフセット更新
        bg0.set_scroll_pos((bg_offset, 0));
        // BG のタイル状態更新
        bg0.set_tile(vram, (bgx, bgy), tileset, tiles::bg.tile_settings[tile_id]);
        bg0.commit(&mut vram);
        droid_object
            // スプライトの位置更新
            .set_position((dx, dy))
            // スプライト画像更新
            .set_sprite(object.sprite(sprite_for_char(ch)));;
        object.commit();
    }
}
```

<div class="src-block-caption">
  <span class="src-block-number">ソースコード 3</span>:
  agb を使った GBA プログラムの概要
</div>

実際のコードは以下:

-   <https://github.com/yewton/jumpingdroidr>
-   <https://github.com/yewton/jumpingdroidr2>


## 成果物 {#成果物}

[mGBA](https://mgba.io/) で jumpingdroidr2 を実行している様子が以下。

BG1 のオフセットとタイルが更新される様子を併せて収録している。
ドロイド君の進行方向にステージを書き足していっている様子が観察出来る。

{{< video src="Screencast 2024-05-11 23:37:55.mp4" controls="yes" >}}


## 終わりに {#終わりに}

普段の開発とは全く異なる考え方が必要になるため面白い。

今回は GBA ソフトの移植を目的とした為に、Rust 自体の学習は必要最低限しかやらなかった。

とっかかりは出来たので、今後細く長くやっていきたい。

agb のリポジトリには例も豊富に提供されており、とても参考になる。

-   <https://github.com/agbrs/agb/tree/master/agb/examples>
-   <https://github.com/agbrs/agb/tree/master/examples>

    その他 GBA 開発関連の情報は [Home | gbadev](https://gbadev.net/) にまとまっている。
