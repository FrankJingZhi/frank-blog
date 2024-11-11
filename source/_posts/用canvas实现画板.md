---
title: 使用HTML5 Canvas创建个性化的画板小工具
date: 2024-7-22 14:50:51
tags: [canvas]
---

### 前言

最近在学习 canvas，顺手实现了一个画板的小工具，分享记录一下。

只要包含的功能点如下：

1. 粗线条
2. 细线条
3. 保存签名
4. 切换颜色
5. 橡皮擦
6. 清空画布

简单的效果演示：

![](/images/canvas-draw.gif)

### 正文

#### HTML 结构设计

首先，我们需要定义页面的基本结构。在这个例子中，我们创建了一个宽度为 600px，高度为 400px 的`<canvas>`元素作为画板，并添加了一系列按钮以实现不同的绘画功能。

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>个性化画板</title>
  </head>
  <body>
    <canvas id="c1" width="600" height="400"></canvas>
    <hr />
    <button id="boldBtn">粗线条</button>
    <button id="thinBtn">细线条</button>
    <button id="saveBtn">保存签名</button>
    <input type="color" id="colorBtn" />
    <button id="clearBtn">橡皮擦</button>
    <button id="nullBtn">清空画布</button>
  </body>
</html>
```

#### JavaScript 逻辑实现

接下来是实现画板的核心功能——绘图逻辑。通过监听鼠标事件，我们可以实现在画布上自由绘制的功能。

```javascript
// 获取画布元素和上下文
const c1 = document.getElementById('c1')
const ctx = c1.getContext('2d')
ctx.lineJoin = 'round'
ctx.lineCap = 'round'

// 获取按钮元素
const boldBtn = document.getElementById('boldBtn')
const thinBtn = document.getElementById('thinBtn')
const saveBtn = document.getElementById('saveBtn')
const colorBtn = document.getElementById('colorBtn')
const clearBtn = document.getElementById('clearBtn')
const nullBtn = document.getElementById('nullBtn')

// 绘制状态控制
let isDraw = false

// 鼠标按下事件
c1.onmousedown = function (e) {
  isDraw = true
  const x = e.pageX - c1.offsetLeft
  const y = e.pageY - c1.offsetTop
  ctx.beginPath()
  ctx.moveTo(x, y)
}

// 鼠标抬起事件
c1.onmouseup = function () {
  isDraw = false
  ctx.closePath()
}

// 鼠标离开事件
c1.onmouseleave = function () {
  isDraw = false
  ctx.closePath()
}

// 鼠标移动事件
c1.onmousemove = function (e) {
  if (isDraw) {
    const x = e.pageX - c1.offsetLeft
    const y = e.pageY - c1.offsetTop
    ctx.lineTo(x, y)
    ctx.stroke()
  }
}
```

#### 功能增强与用户体验优化

为了让用户能够更方便地调整线条样式和颜色，我们为每个按钮添加了相应的点击事件处理程序。

- **线条粗细切换**：通过更改`lineWidth`属性值，用户可以轻松选择绘制线条的粗细。
- **颜色选择器**：用户可以通过颜色选择器动态设置绘制线条的颜色。
- **橡皮擦功能**：通过设置`globalCompositeOperation`为`destination-out`，实现擦除效果。
- **清空画布**：一键清除所有内容，重置画布。
- **保存签名**：将当前画布内容转换为 Data URL 格式，并展示为图片，便于用户保存或分享。

```javascript
// 粗线条按钮点击事件
boldBtn.onclick = function () {
  ctx.lineWidth = 5
  ctx.globalCompositeOperation = 'source-over'
}

// 细线条按钮点击事件
thinBtn.onclick = function () {
  ctx.lineWidth = 1
  ctx.globalCompositeOperation = 'source-over'
}

// 颜色选择器变化事件
colorBtn.onchange = function () {
  ctx.strokeStyle = this.value
}

// 橡皮擦按钮点击事件
clearBtn.onclick = function () {
  ctx.globalCompositeOperation = 'destination-out'
  ctx.lineWidth = 5 // 设置橡皮擦的宽度
}

// 清空画布按钮点击事件
nullBtn.onclick = function () {
  ctx.clearRect(0, 0, c1.width, c1.height)
}

// 保存签名按钮点击事件
saveBtn.onclick = function () {
  const url = c1.toDataURL()
  const img = new Image()
  img.src = url
  document.body.appendChild(img)
}
```

#### API 解释

- **`globalCompositeOperation`**：这是一个属性，用于设置绘制新图形时如何与已有的图形进行混合。可用的值有非常多，我这里只列举文中用到的两个，其他的大家可以在 mdn 自行查找：

  - `source-over`：新的图形覆盖在已有图形之上（默认值）。
  - `destination-out`：新的图形会擦除已有图形的一部分，常用于实现橡皮擦效果。

- **`toDataURL()`**：这是一个方法，用于将当前画布的内容转换为一个数据 URL，通常是一个 Base64 编码的字符串。这个 URL 可以直接用于 `<img>` 标签的 `src` 属性，也可以用于下载图像文件。

#### 结语

通过上述步骤，我们就成功地构建了一个具备基本功能的在线画板小工具。这个项目帮我更好地理解 HTML5 Canvas 的工作原理，Canvas 才是前端动画的最正解，简单又高效！
