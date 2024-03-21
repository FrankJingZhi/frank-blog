---
title: 对tailwind css的理解和思考
date: 2024-03-20 11:29:24
tags: ['Tailwind CSS', 'rem', '多端适配']
---

## 前言

今天来跟大家聊一下再使用 tailwind css 时的一些感悟。

## 正文

### tailwind css 是什么

官网的介绍是这样的：

> Rapidly build modern websites without ever leaving your HTML.
> A utility-first CSS framework packed with classes like flex, pt-4, text-center and rotate-90 that can be composed to build any design, directly in your markup.

你不需要脱离你的 HTML 来编写网站。

一个实用程序优先的 CSS 框架，包含 `flex pt-4 text-center rotate-90` 等类，可以直接在标记中组合它们来构建任何设计。

也就是说我们可以通过直接添加 `flex pt-4 text-center rotate-90` 等相应的类名就可以，不需要再为页面单独编写 css 文件了。

这听起来很棒！

### 为什么需要 tailwind css

在我的使用感受上来说，`tailwind css`解决了我的以下几个痛点：

1. 不需要再为`css`提供单独的文件了，这样我就不用在`html`和`css`之间来回穿梭了。
2. `tailwind css`使得`css`原子化，这就像一个工具库，我只需要调用其中的`api`就行了。只不过这个工具库是`css`工具库。
3. 它的单位是`rem`，这使得我在做多端适配和页面等比例缩放的时候非常方便。

### 如何使用 tailwind css

附上![Tailwind CSS官网](https://tailwindcss.com/docs/installation)的路径。官网的操作手册很详细，我就不多 BB 了。

![](/images/funny-1.webp)

安装完成后，打开我们的页面和代码看一下

![](/images/tailwindcss-1.png)

![](/images/tailwindcss-2.png)

生效了，而且也只打包进了我们使用到的几个样式，而且多次使用同一种样式，`output.css`种也只是打包进了一份。

我认为这是`tailwind css`最有价值的点之一。

### 业务场景一：多端适配

附上![官网自适应](https://tailwindcss.com/docs/responsive-design)的链接。

![](/images/tailwindcss-9.png)

`tailwind css`拆分出了各种尺寸的适配，我们只需要在类前添加对应的标识即可。如下：

```html
<!-- 在1024px以上的屏幕中使用w-48，在768px - 1024px之间的屏幕中使用w-32 -->
<img class="w-16 md:w-32 lg:w-48" src="..." />
```

看一下官网的案例效果：

![](/images/tailwindcss-7.gif)

我认为这是`tailwind css`最有价值的点之二。

### 业务场景二：等比例缩放

首先，我们思考这样一个问题：

> 在一个项目内同时包含 pc 端页面 A、ipad 端页面 B、移动端页面 C。A、B、C 三个页面的设计图宽度分别为 1024px、768px、375px。设计师期望在可视区域宽度为 1024px 的 pc 设备上可以 1:1 显示页面 A（实际尺寸:设计图尺寸），而在其他宽度的 pc 设备上对页面 A 做等比缩放。同样的，还希望在可视区域宽度为 768px 的 ipad 设备上可以 1:1 显示页面 B，而在其他宽度的 ipad 设备上对页面 B 做等比缩放。页面 C 同理。

要解决这个问题，我们要知道一个知识点：

`rem`的大小是随页面根元素（通常是`html`）的大小变化的。可以简单理解为`元素字体大小 = 元素rem * 根元素px`。举例说明：

```html
<html style="fontSize: 32px">
  <p style="fontSize: 1rem;">Hello, World!</p>
</html>
```

此时，p 元素的字体大小转换为 px 为`1rem * 32px = 32px`。

明白这个知识点后，就很简单了，因为`tailwind css`使用的单位都是`rem`，我们只需要在页面变化的时候，修改根元素大小就行了。

```js
// 根据页面尺寸，等比例缩放页面
function setRem() {
  const baseSize = 16 // 基准大小，取16px
  const width = document.documentElement.clientWidth // 当前页面宽度
  let scale = 1 // 缩放比例
  let currentDevice = 'lg' // 当前设备是哪个尺寸
  const deviceMap = {
    lg: width >= 1024,
    md: width >= 768 && width < 1024,
    sm: width < 375,
  }
  for (const key in deviceMap) {
    if (deviceMap[key]) {
      currentDevice = key
    }
  }
  const DeviceEnum = {
    lg: 1024,
    md: 768,
    sm: 375,
  }
  scale = width / DeviceEnum[currentDevice]
  document.documentElement.style.fontSize = baseSize * scale + 'px'
}
setRem()
window.onresize = setRem
```

看下效果：

![](/images/tailwindcss-7.gif)

这是![我的demo仓库](https://github.com/FrankJingZhi/tailwind-css-demo)，大家可以下载看一下。

我认为这是`tailwind css`最有价值的点之三。

## 后序

这是今天分享的内容，感谢观看
