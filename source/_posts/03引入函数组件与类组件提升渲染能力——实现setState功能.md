---
title: 03引入函数组件与类组件提升渲染能力——实现setState功能
date: 2024-01-11 16:55:30
tags: [React]
categories: [从0到1实现React核心功能]
---

# 前言

> 这是`React`的深度学习系列的第二篇，这个系列是通过从 0 到 1 一步步手写`React`实现自己的`mini-react`，以此来达到阅读源码、学习源码的目的。

> PS：本章节实现的内容为类组件版本的 React，函数组件版本的会在后续章节中实现 🌟

本章节我们主要实现如下功能：

1. 分析 setState 的更新原理
2. 实现 setState

# 正文

## 分析 setState 的更新原理

我们一起来看这个图例，思考一下如何实现这个功能，再对应到 react 中是如何实现的：
![](/images/mini-react-02-setState.png)

如果我们想要实现这个能力，目前有两种方式：

1. 对油漆桶装一个监听器，每当粉刷工发现油漆桶中的油漆颜色变化时，就进行粉刷。

2. 油漆装填工更换完油漆后，油漆桶再通知粉刷工油漆更换了，你需要重新进行粉刷了

这两种实现方案也是目前前端主流框架进行数据驱动视图更新的方案。方案一对应 vue，方案二对应 react。

也就是说我们调用 setState 更新数据之后，setState 会调用 createDOM 重新绘制 dom，并挂载到页面上。（这里我们先不考虑 dom diff，后面会实现）。明白原理之后，我们来实现它。

## 实现 setState

### 实现 setState 的单个更新功能

通过上面的分析，我们知道：

1. 我们需要有一个 setState 方法来接受数据
2. 我们需要有一个 update 方法来调用 createDOM 实现视图更新

```js
// 类组件
export class Component {
  // 注明这是一个类组件，区别于函数组件
  static IS_CLASS_COMPONENT = true
  constructor(props) {
    this.props = props
    this.state = {}
  }
  setState(partialState) {
    // 接收数据
  }
  update() {
    // 获取新的虚拟dom
    // 根据新的虚拟dom更新真实dom
    // 将真实dom挂载到容器上
  }
}
```

因为数据处理本身是一个非常复杂的事情，所以我们单独提取一个类来做数据状态管理。这个类需要做的事情就是，`合并数据，并将合并好的数据赋给组件实例的state，然后调用组件实例的update方法通知页面更新`

```js
// 管理状态
class Updater {
  constructor(classComponentInstance) {
    // 需要更新的组件实例
    this.classComponentInstance = classComponentInstance
    // 存储需要更新的状态
    this.pendingStates = []
  }
  addState(partialState) {
    // 将需要更新的状态存储起来
    this.pendingStates.push(partialState)
    // 预处理更新
    this.preHandleForUpdate()
  }
  preHandleForUpdate() {
    // 数据更新
    this.launchUpdate()
  }
  launchUpdate() {
    const { classComponentInstance, pendingStates } = this
    if (pendingStates.length === 0) return
    // 合并状态数据
    classComponentInstance.state = pendingStates.reduce(
      (preStates, newState) => {
        return { ...preStates, ...newState }
      },
      classComponentInstance.state
    )
    // 清空状态
    this.pendingStates.length = 0
    // 更新组件
    classComponentInstance.update()
  }
}
```

然后我们来补充一下 setState 方法和 update 方法

```js
export class Component {
  // 注明这是一个类组件，区别于函数组件
  static IS_CLASS_COMPONENT = true
  constructor(props) {
    this.props = props
    this.updater = new Updater(this)
    this.state = {}
  }
  setState(partialState) {
    // 合并数据
    // 重新渲染进行更新
    this.updater.addState(partialState)
  }
  update() {
    // 获取旧的虚拟dom和新的虚拟dom
    // 将旧的dom删除
    // 将新的dom挂载到容器上
    let oldVNode = this.oldVNode // TODO：让类组件拥有一个oldVNode属性保存类组件实例对应的虚拟dom
    let oldDOM = findDOMByVNode(oldVNode) // TODO：让真实dom保存到oldVNode上
    let newVNode = this.render()
    updateDOMTree(oldDOM, newVNode) // TODO：更新dom
    this.oldVNode = newVNode
  }
}
```

这里我们有三个 TODO 项需要做，我们一个一个处理：

1. TODO：让类组件拥有一个 oldVNode 属性保存类组件实例对应的虚拟 dom

   这个东西我们就可以在创建完虚拟 dom 时绑定，也就是在 createDOM 中绑定

   ```js
   function createDOM(VNode) {
     // ...此处省去无关的代码
     // 处理属性值
     setPropsFromDOM(dom, props)
     // 将虚拟dom和dom绑定
     VNode.dom = dom
     return dom
   }
   ```

2. TODO：让真实 dom 保存到 oldVNode 上

   createDOM 中已经绑定了，为了保持函数职责单一，我们提供一个新的方法来处理

   ```js
   // 根据虚拟dom找到dom
   export function findDOMByVNode(VNode) {
     if (!VNode) return
     if (VNode.dom) return VNode.dom
   }
   ```

3. TODO：更新 dom

   ```js
   // 更新dom树
   export function updateDOMTree(oldDOM, newVNode) {
     // 找到当前dom的父节点
     const parentNode = oldDOM && oldDOM.parentNode
     // 移除掉旧的dom
     parentNode.removeChild(oldDOM)
     // 挂载新的dom
     parentNode.appendChild(createDOM(newVNode))
   }
   ```

好了，到目前为止，我们就实现了 setState 的单个更新功能，我们来尝试一下。

### setState 的单个更新功能尝试

我们来定义一个 name 值为 React，3 秒后将 name 更新为 mini-react

首先，我们修改一下`src/index.js`

```js
class MyComponent extends React.Component {
  constructor(props) {
    super(props)
    this.state = {
      name: 'React',
    }
  }
  render() {
    return <div>Hello, {this.state.name}!</div>
  }
}

ReactDOM.render(<MyComponent />, document.getElementById('root'))
```

因为我们这里还没有实现事件委托机制，所以我们就在类实例初始化完成后，使用 setTimeout 调用 setState 进行更新。

```js
// 处理类组件
function getClassDOM(VNode) {
  const { type, props, ref } = VNode
  const instance = new type(props)
  const renderVNode = instance.render()
  // 将虚拟dom保存到类组件实例上，方便后续更新
  instance.oldVNode = renderVNode
  // 将ref和dom绑定
  ref && (ref.current = instance)
  // ✨TODO: 需要删除的代码 start
  setTimeout(() => {
    instance.setState({ name: 'mini-react' })
  }, 3000)
  // ✨TODO: 需要删除的代码 end
  if (!renderVNode) return null
  return createDOM(renderVNode)
}
```

我们看一下页面是否正常渲染

![](/images/mini-react-02-setState-single-demo.gif)

OK，单个的渲染功能完成 ✨

### setState 的批量更新

我们代码中经常会有频繁调用 setState 的情况，所以我们需要区分处理单个更新和批量更新的情况

```js
// Component.js
// 批量更新队列 - 管理Updater
export let UpdaterQueue = {
  isBatch: false,
  updaters: new Set(),
}

// 执行更新队列
export function flushUpdaterQueue() {
  // 遍历更新队列，执行更新
  UpdaterQueue.updaters.forEach((updater) => {
    updater.launchUpdate()
  })
  // 清空队列
  UpdaterQueue.updaters.clear()
  // 关闭批量更新
  UpdaterQueue.isBatch = false
}

// 管理状态
class Updater {
  // ...省略无关代码
  preHandleForUpdate() {
    if (UpdaterQueue.isBatch) {
      // 批量更新
      UpdaterQueue.updaters.add(this)
    } else {
      // 单个更新
      this.launchUpdate()
    }
  }
}
```

### setState 的批量更新功能尝试

跟单个更新一样，我们目前也是只能在 getClassDOM 方法中再加一个 setTimeout

```js
// TODO: 需要删除的代码 start
setTimeout(() => {
  instance.setState({ name: 'mini-react1' })
}, 3000)
setTimeout(() => {
  instance.setState({ name: 'mini-react2' })
}, 6000)
// TODO: 需要删除的代码 end
```

我们看下运行结果
![](/images/mini-react-02-setState-batch-demo.gif)

OK，没问题

最后看一下`src/Component.js`完整的代码

```js
import { findDOMByVNode, updateDOMTree } from './react-dom'

// 批量更新队列 - 管理Updater
export let UpdaterQueue = {
  isBatch: false,
  updaters: new Set(),
}
// 执行更新队列
export function flushUpdaterQueue() {
  // 遍历更新队列，执行更新
  UpdaterQueue.updaters.forEach((updater) => {
    updater.launchUpdate()
  })
  // 清空队列
  UpdaterQueue.updaters.clear()
  // 关闭批量更新
  UpdaterQueue.isBatch = false
}
// 管理状态
class Updater {
  constructor(classComponentInstance) {
    // 需要更新的组件实例
    this.classComponentInstance = classComponentInstance
    // 存储需要更新的状态
    this.pendingStates = []
  }
  addState(partialState) {
    // 将需要更新的状态存储起来
    this.pendingStates.push(partialState)
    // 预处理更新
    this.preHandleForUpdate()
  }
  preHandleForUpdate() {
    if (UpdaterQueue.isBatch) {
      // 批量更新
      UpdaterQueue.updaters.add(this)
    } else {
      // 单个更新
      this.launchUpdate()
    }
  }
  launchUpdate() {
    const { classComponentInstance, pendingStates } = this
    if (pendingStates.length === 0) return
    // 合并状态数据
    classComponentInstance.state = pendingStates.reduce(
      (preStates, newState) => {
        return { ...preStates, ...newState }
      },
      classComponentInstance.state
    )
    // 清空状态
    this.pendingStates.length = 0
    // 更新组件
    classComponentInstance.update()
  }
}

// 类组件
export class Component {
  // 注明这是一个类组件，区别于函数组件
  static IS_CLASS_COMPONENT = true
  constructor(props) {
    this.props = props
    this.updater = new Updater(this)
    this.state = {}
  }
  setState(partialState) {
    // 合并数据
    // 重新渲染进行更新
    this.updater.addState(partialState)
  }
  update() {
    // 获取旧的虚拟dom和新的虚拟dom
    // 将旧的dom删除
    // 将新的dom挂载到容器上
    let oldVNode = this.oldVNode // 让类组件拥有一个oldVNode属性保存类组件实例对应的虚拟dom
    let oldDOM = findDOMByVNode(oldVNode) // 让真实dom保存到oldVNode上
    let newVNode = this.render()
    updateDOMTree(oldDOM, newVNode)
    this.oldVNode = newVNode
  }
}
```

# 后序

OK，到这里 setState 的功能就算实现了，因为我们没有实现事件委托系统，什么时候开始批量更新，我们就留到下一篇事件委托机制中一起演示吧。🎉
