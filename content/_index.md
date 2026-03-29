---
title: yewton.net
type: landing

sections:
  - block: hero
    content:
      title: yewton.net
      text: 'yewton が <a href="/blog/" class="underline">ブログ</a> を書いたり <a href="/cv/" class="underline">オープン職務経歴書</a> を公開したりしています。'
    design:
      no_padding: true
      css_class: dark
      css_style: "background-image: linear-gradient(rgba(0,0,0,0.5), rgba(0,0,0,0.5)), url('/media/home-bg.jpg'); background-size: cover; background-position: top;"
      spacing:
        padding: ['80px', '0', '80px', '0']

  - block: collection
    id: posts
    content:
      title: 最近の投稿
      subtitle: ''
      text: ''
      count: 5
      offset: 0
      order: desc
      filters:
        folders:
          - post
    design:
      view: card
---
