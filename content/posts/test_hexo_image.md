---
title: 测试Hexo图片
date: 2014-04-18 21:44:32
tags: [test, hexo, image, audio, video]
---

### 图片
![Something][GoodImage]

[GoodImage]:http://t.williamgates.net/thumb-0DD4_53C7517B.jpg "GoodGirl"

`<img src="http://t.williamgates.net/thumb-0DD4_53C7517B.jpg" style="width: 500">`


hahaha

下一步就是拿flask写个图床放BAE上,搞定!!

### 音频

<embed src="http://www.xiami.com/widget/0_3515679/singlePlayer.swf" type="application/x-shockwave-flash" width="257" height="33" wmode="transparent"></embed>

html的方法 暴力...

空这么大一块是由于 video-containter 的 css 造成的, 想修改就得看源码了, 留到以后用吧. 或者用 HTML5 解决.这个方法不错.

``` [html]
<embed src="http://www.xiami.com/widget/0_3515679/singlePlayer.swf" type="application/x-shockwave-flash" width="257" height="33" wmode="transparent"></embed>
```

### 视频

<iframe height=498 width=510 src="http://player.youku.com/embed/XMzEwNTgwMjI4" frameborder=0 allowfullscreen></iframe>

``` [html]
<iframe height=498 width=510 src="http://player.youku.com/embed/XMzEwNTgwMjI4" frameborder=0 allowfullscreen></iframe>
```

<!--
```[lang]
code
```
-->
