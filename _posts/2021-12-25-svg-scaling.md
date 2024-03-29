---
layout: mypost
title: 像图片一样缩放SVG
categories: [其他]
---

最近在做一个 OCR 相关的功能，需要高亮圈出文字的范围。接口调用的是腾讯的表格识别，结果里面已经返回了 polygon 的坐标点信息，所以很容易就想到在前端使用 svg 绘制 polygon 然后覆盖在在图片上。

结果发现只有在图片 1:1 展示的时候，框选的范围是正确的。当图片做了缩放后，同步为 SVG 设置为图片的相同宽度后，发现 polygon 并未按照想象中的效果等比缩放。

![demo](01.png)

### 方案 1

既然无法达到和图片一样的缩放效果，可以考虑把 svg 放到 img 的 src 里面去展示。svg 的宽高和图片一致，通过调整 img 的宽度和原始图片的宽度一致就达到了同步的缩放效果

```html
<div class="left">
  <img class="origin" style="width: 100%; height: auto" />
  <img class="svg" style="position: absolute; left: 0; top: 0; width: 100%; height: auto" />
</div>
```

```js
function initSVG(points) {
  var img = new Image()
  img.onload = function () {
    let svgDomStr = `<svg xmlns="http://www.w3.org/2000/svg" width="${img.width}" height="${img.height}">`
    svgDomStr += `<polygon points="${points}" style="fill: none; stroke: #ff0000; stroke-width: 2" />`
    svgDomStr += `</svg>`
    let url = 'data:image/svg+xml;base64,' + btoa(svgDomStr)
    $('img.svg').attr('src', url)
  }
  img.src = $('.left img').attr('src')
}
```

另外除了 base64 外，还可以通过下面的方式生成一个虚拟的 URL 放到 src 内，也是一样的效果

```js
const blob = new Blob([svgDomStr], {
  type: 'image/svg+xml'
})
$('img.svg').attr('src', URL.createObjectURL(blob))
```

### 方案 2

仔细阅读了 SVG 的文档后，发现自己对 svg 的宽高理解有误。在 svg 系统中，画布的长和宽其实是无限的，viewBox 属性来说明那个区域是可以被看到的，preserveAspectRatio 属性定义 viewBox 在 width 和 height 区域内的表现

所以 svg 内的坐标都应该以 viewBox 定义为范围为基准，而不是以 width/height 为基准

preserveAspectRatio 是和 viewBox 搭配使用的，单独使用无效，默认值是`xMidYMid meet`,代表位置和缩放效果。位置共有`[x,y][Min, Mid, Max]`九种组合效果，缩放有`[meet, slice]`等比缩放和剪切两种

preserveAspectRatio 还有一种取值就是 `none`,这种情况下就和 img 效果一样了，在 width/height 填充满区域并拉伸缩放

如下在第二个图形可以看到，在不设置 viewBox 的情况下，width 改变，圆的位置及形状未发生任何的改变，在设置了 viewBox 属性后，位置才变得可控

![02](02.png)

综上，只需要设置 viewBox 和图片的真实宽高一样，同时设置 preserveAspectRatio 允许拉伸缩放即可

```html
<div class="left">
  <div class="container" style="position: relative">
    <img class="origin" style="width: 100%; height: auto" />
    <svg
      xmlns="http://www.w3.org/2000/svg"
      preserveAspectRatio="none"
      width="100%"
      height="100%"
      style="position: absolute; left: 0; top: 0"
    >
      <polygon style="fill: none; stroke: #ff0000; stroke-width: 2" />
    </svg>
  </div>
</div>
```

```js
function initSVG(points) {
  var img = new Image()
  img.onload = function () {
    let svgDOM = $('svg')[0]
    svgDOM.setAttribute('viewBox', `0 0 ${img.width} ${img.height}`)
    svgDOM.querySelector('polygon').setAttribute('points', points)
  }
  img.src = $('.left img').attr('src')
}
```

### 拓展

实现一个图片剪裁展示的效果，比如雪碧图，一般写法如下

![03](03.png)

上面的缺点同样很明显，宽高是定死的，无法自适应，通过SVG同样也可以解决这个问题

![04](04.png)

### 总结

一个小坑，不要使用 jquery 的 attr 来操作 svg 属性，比如用 jquery 设置 viewBox 属性，发现设置到 dom 上变成了小写，导致不生效

在写 svg 的时候，标准的做法是把 width，height，viewBox 都定义出来，viewBox 的范围就是 width，height 的范围。

### 参考

[SVG之ViewBox](https://segmentfault.com/a/1190000009226427)
