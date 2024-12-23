---
title: 性能优化-CLS
date: 2024-9-19 15:51:03
tags: [性能优化, CLS]
---

## 优化 CLS（Cumilative Layout Shift）

### 概念

**CLS 是什么**

通过将视口中可见内容偏移的量与受影响元素移动的距离相结合来衡量内容的不稳定性指标。

**CLS 不佳的最常见原因如下**

1. 图片没有尺寸。
2. 没有尺寸的广告、嵌入内容和 iframe。
3. 动态注入的内容，例如无尺寸的广告、嵌入内容和 iframe。
4. 网络字体。

### 优化方案

1. 对于 img，使用 srcset 属性，并设置 width 和 height 属性

   1. 由于现代浏览器会根据 图片的 width 和 height 属性，您可以通过执行以下操作来防止布局偏移： 在图片上设置这些属性。所有浏览器都会根据元素的现有 width 和 height 属性添加默认宽高比。

   2. aspect-ratio：CSS 属性允许您定义元素框所需的宽高比。这意味着即使父容器或视口大小发生变化，浏览器也会调整元素的尺寸以保持指定的宽高比。指定的宽高比用于计算自动尺寸和其他一些布局功能。至少有一个框的尺寸需要是自动的才能 aspect-ratio 产生效果。

![](/images/cls1.png)

2. 骨架载入，内容渲染前先用颜色类似的内容代替展示

3. 为元素设置宽高

![](/images/cls2.png)

参考内容：[https://web.dev/articles/cls?hl=zh-cn](https://web.dev/articles/cls?hl=zh-cn)
