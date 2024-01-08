---
title: 从0到1实现React核心功能-01初步实现页面渲染
date: 2024-01-08 12:18:40
tags: [React]
categories: [从0到1实现React核心功能]
---

# 前言

> 这个系列是`React`的深度学习系列。通过从 0 到 1 一步步手写`React`实现自己的`mini-react`，以此来达到阅读源码、学习源码的目的。

本章节我们主要实现如下几个功能：

1. 环境搭建
2. 理解虚拟 DOM 与 DOM
3. 分析 React 和 ReactDOM 的职责划分
4. 实现 React.createElement 函数和 ReactDOM.render 函数——理解 jsx 的相关概念及其原理
5. 一起思考几个问题

# 正文

## 环境搭建

1. 首先，我们要搭建一个 React 项目，执行如下命令：

   ```zsh
   npx create-react-app mini-react
   ```

2. 删除无用的文件，保证文件目录如下：
   ![](/images/mini-react-01-catalog.png)

3. 调整以下文件的内容：

   - package.json
     因为我们要实现原始的版本，也就是 React18 以前的版本，所以需要做如下处理

   ```json
     "scripts": {
       "start": "DISABLE_NEW_JSX_TRANSFORM=true react-scripts start"
     },
   ```

   - src/index.js

   ```jsx
   import ReactDOM from 'react-dom'
   import React from 'react'

   ReactDOM.render(
     <div id='box' className='box' style={{ color: 'red' }}>
       hello, react!
       <span>xxx1</span>
       <span>xxx2</span>
     </div>,
     document.getElementById('root')
   )
   ```

   到这里，我们的环境就搭建好了。

## 理解虚拟 DOM 与 DOM

1. 首先，我们来思考一个问题：如果我们用最原始的方式修改页面的样式或者 DOM 结构，我们该怎么做？比如当我们点击某个按钮时，要给 root 节点添加一个子元素——span 标签，内容为 xxx

   > 我们接下来带着这个问题继续往下看

2. 什么是 dom？

   [mdn](https://developer.mozilla.org/zh-CN/docs/Web/API/Document_Object_Model/Introduction)中给出如下解释：
   ![](/images/mini-react-01-dom.png)

   如果不是很理解，也没关系，这里我们可以通俗的理解为页面上的一个个 HTML 文本节点。

   以我们搭建好的项目中 root 节点为例，我们来看一下这个 DOM 中都包含哪些内容：

   在浏览器控制台中输入如下代码：

   ```javascript
   let dom = {}
   const root = document.getElementById('root')
   for (const key in root) {
     dom[key] = root[key]
   }
   console.log('dom', dom)
   ```

   我们可以看到如下输出：
   ![](/images/mini-react-01-dom-object.png)

   这还只是显示了很少的一部分，完整的会更多。我们每一次操作 DOM 都需要处理如此复杂的一个对象，可想而知，这个代价有多大。

3. 什么是虚拟 DOM？

   [React 官网](https://legacy.reactjs.org/docs/faq-internals.html)中给出如下解释：
   ![](/images/mini-react-01-virtual-dom.png)

   emmm，看着还是很抽象，没关系，我们来看一个例子。

   在`src/index.js`中打印一下我们渲染的内容：

   ```javascript
   console.log(
     <div id='box' className='box' style={{ color: 'red' }}>
       hello, react!<span>xxx1</span>
       <span>xxx2</span>
     </div>
   )
   ```

   控制台输出如下：
   ![](/images/mini-react-01-virtual-dom-object.png)

   这里面的属性内容明显就少很多，相比真实 DOM，操作起来就会方便很多，性能也会有很大的提升。

   这个对象官方叫`ReactElement`，这里我们可以顺便解释一下对象中的属性含义：

   ```javascript
   export type ReactElement = {
     // 用于辨别ReactElement对象
     $$typeof: any,

     // 内部属性
     type: any, // 表明其种类
     key: any, // key属性在reconciler阶段会用到, 目前只需要知道所有的ReactElement对象都有 key 属性(且其默认值是 null, 这点十分重要, 在 diff 算法中会使用到).
     ref: any,
     props: any,

     // 以下对象用不到，可以暂时忽略
     // ReactFiber 记录创建本对象的Fiber节点, 还未与Fiber树关联之前, 该属性为null
     _owner: any,

     // __DEV__ dev环境下的一些额外信息, 如文件路径, 文件名, 行列信息等
     _store: { validated: boolean, ... },
     _self: React$Element<any>,
     _shadowChildren: any,
     _source: Source,
   }
   ```

   到这里，DOM 和虚拟 DOM 就说完了，我们再回到最开始的问题，分别用 DOM 和虚拟 DOM 来完成这个功能，哪一个更简单，更节省性能呢？

   想必大家心中也有了答案，那就是用虚拟 DOM。

## 分析 react 包和 react-dom 包的职责划分

1. react：专注于组件和 api 的暴露，是跨平台的
2. react-dom：专注于将渲染好的组件渲染到哪里去，我们看源码文件可以知道，这个包分为 react-dom/client 包和 react-dom/server 包，也就是分别对性 web 端和服务端

## 理解 jsx 的相关概念及其原理（手写部分）✨

我们来实现最基本的功能，将`src/index.js`中实现的功能改为我们自己的实现。

### 实现 React.createElement 函数

首先是引入 React

```js
import React from 'react'
```

那么问题就来了，我们发现它只是单纯的引入了 React 但是并没有在代码中使用。那我们把这段代码删除掉看一下，发现页面报错了![](/images/mini-react-01-not-import-react.png)

这是为什么呢？这就引申出了一道 React 经典的面试题：React18 版本以前为什么我们在使用 jsx 的时候必须要引入 React？

这一点我们可以通过转换后的代码来看，在[这里](https://babeljs.io/repl)输入 jsx，babel 会帮我们转换成浏览器真正运行时的代码。（PS：通过切换 React Runtime 可以改变当前使用的 React 版本，Classic 代表 React18 以前的版本，Automatic 代表 React18 以后的版本）

在 react18 以前的版本，jsx 语法解析完成后的代码是通过 react.createElement 函数来调用执行的，所以文件中必须要引入 react，即使它表面上没有被引用
![](/images/before-react18.png)

react18 版本以后，jsx 语法被单独提取了出来，成为一个独立的函数，与 react 包无关了，所以不需要额外引用 react 了
![](/images/after-react18.png)

也就是说 createElement 函数是负责将 babel 转换后的内容再次处理成 VNode 的结构，好，明白了这一点我们接下来就来实现 createElement 函数。

新建`react.js`和`util.js`文件，文件内容如下（我们直接写出来跟大家解释）：

```js
// util.js
export const REACT_ELEMENT = Symbol('react.element') // 标识VNode

// react.js
import { REACT_ELEMENT } from './utils'
function createElement(type, properties, children) {
  // 这几个值暂时没有用到，先不关注
  let key = properties.key || null
  let ref = properties.ref || null
  ;['__self', '__source', 'key', 'ref'].forEach((key) => delete properties[key])

  let props = { ...properties }
  if (arguments.length > 3) {
    // 如果传入的参数超过3个，就代表这个元素中还有子元素，我们把所有子元素放到数组里
    props.children = Array.prototype.slice.call(arguments, 2)
  } else {
    // 否则我们就直接使用第三个参数
    props.children = children
  }
  // 最终抛出去VNode
  return {
    $$typeof: REACT_ELEMENT,
    type,
    props,
    key,
    ref,
  }
}

const React = {
  createElement,
}

export default React
```

我们注释掉`src/index.js`其他代码，只保留如下代码，看下控制台打印结果是否和之前的一致

```js
import React from './react'
console.log(
  <div id='box' className='box' style={{ color: 'red' }}>
    hello, react!<span>xxx1</span>
    <span>xxx2</span>
  </div>
)
```

![](/images/mini-react-01-createElement.png)

我们可以看到除去暂时不需要关注的内容，其他关键内容都是一致的。

### 实现 ReactDOM.render 函数

ReactDOM.render 函数目前来看就负责两件事：

1. 将虚拟 DOM 转换成真实 DOM
2. 将虚拟 DOM 挂在到容器上

新建`react-dom.js`文件，实现内容如下：

```js
import { REACT_ELEMENT } from './utils'

function render(VNode, containerDOM) {
  // 将虚拟DOM转换成真实DOM
  // 将虚拟DOM挂在到容器上
  mount(VNode, containerDOM)
}

function mount(VNode, containerDOM) {
  // 将虚拟DOM转换成真实DOM
  let newDOM = createDOM(VNode)
  // 将虚拟DOM挂在到容器上
  newDOM && containerDOM.appendChild(newDOM)
}

// 处理子元素数组
function mountArray(VNodes, containerDOM) {
  if (!Array.isArray(VNodes)) return
  VNodes.forEach((vnode) => {
    if (typeof vnode === 'string') {
      containerDOM.appendChild(document.createTextNode(vnode))
    } else {
      mount(vnode, containerDOM)
    }
  })
}

// 处理属性
function setPropsFromDOM(dom, VNodeProps = {}) {
  if (!VNodeProps) return
  for (let key in VNodeProps) {
    if (key === 'children') continue
    if (key === 'style') {
      for (let styleKey in VNodeProps.style) {
        dom.style[styleKey] = VNodeProps.style[styleKey]
      }
    } else if (/^on[A-Z].*/.test(key)) {
      // TODO: 事件绑定
    } else {
      dom[key] = VNodeProps[key]
    }
  }
}

function createDOM(VNode) {
  const { type, props } = VNode
  let dom
  // 创建元素
  if (type && VNode.$$typeof === REACT_ELEMENT) {
    dom = document.createElement(type)
  }
  // 处理子元素
  if (props) {
    if (typeof props.children === 'object' && props.children.type) {
      // 如果children是对象&&存在type，那么说明children是个子元素标签，我们直接把它挂载到父元素上
      mount(props.children, dom)
    } else if (Array.isArray(props.children)) {
      // 如果children是一个数组，那么说明children有并列的兄弟元素，我们需要遍历处理兄弟元素
      mountArray(props.children, dom)
    } else if (typeof props.children === 'string') {
      // 如果children是个string，那么说明children就是一个单纯的文本内容，那么我们直接将文本挂载到父元素上
      dom.appendChild(document.createTextNode(props.children))
    }
  }
  // 处理属性值
  setPropsFromDOM(dom, props)
  return dom
}

const ReactDOM = {
  render,
}
export default ReactDOM
```

我们将`src/index.js`内容引用修改为我们自己的：

```js
import ReactDOM from './react-dom'
import React from './react'

console.log(
  <div id='box' className='box' style={{ color: 'red' }}>
    hello, react!<span>xxx1</span>
    <span>xxx2</span>
  </div>
)

ReactDOM.render(
  <div id='box' className='box' style={{ color: 'red' }}>
    hello, react!<span>xxx1</span>
    <span>xxx2</span>
  </div>,
  document.getElementById('root')
)
```

我们看一下页面是否正常渲染：
![](/images/mini-react-01-reactdom-render.png)

页面正常渲染，OK，我们的基本功能手写完成 ✅

## 思考题

我们留下两个问题一起思考，下篇文章给出解答。

1. 如果不用 jsx，如何用 react 在页面上渲染"Hello, react!"

2. 从源代码`ReactDOM.render(<div id='box' className='box' style={{color: 'red'}}>hello, react!<span>xxx1</span><span>xxx2</span></div>, document.getElementById('root'))`到页面显示，经历了那些关键步骤

# 后序

本篇分享结束，欢迎大家一起讨论，我们下一篇见 👋 ～
