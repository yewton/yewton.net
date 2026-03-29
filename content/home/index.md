+++
lastmod = 2020-01-13T06:56:57+09:00
# Homepage is now a standard page with blocks.

[[blocks]]
  # Hero block
  blox = "hero"
  [blocks.content]
    title = "yewton.net"
    text = "yewton が [ブログ](/blog/) を書いたり [オープン職務経歴書](/cv/) を公開したりしています。"

  [blocks.design.background]
    gradient_colors = ["#4bb4e3", "#2b94c3"]
    image = "home-bg.jpg"
    image_darken = 0.6
    image_size = "cover"
    image_position = "center"
    image_parallax = true
    text_color_light = true

[[blocks]]
  # Recent posts block
  blox = "collection"
  [blocks.content]
    title = "最近の投稿"
    page_type = "post"
    count = 5
    order = "desc"
    [blocks.content.filters]
      exclude_featured = false

  [blocks.design]
    view = "compact"
+++
