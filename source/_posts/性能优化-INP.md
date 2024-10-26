---
title: 性能优化-INP
date: 2024-10-26 14:02:34
tags: [性能优化, INP]
---

## 优化 INP（Interaction to Next Paint）

### 概念和事实

> ✨ 注意：Interaction to Next Paint (INP) 是一项稳定的 Core Web Vitals 指标，用于使用 Event Timing API 中的数据评估响应速度。INP 会观察用户与网页进行的所有互动的延迟时间，并报告所有（或几乎所有）互动都低于此值的单个值。INP 较低意味着网页能够始终如一地快速响应所有或绝大多数的用户互动。

**概念**

INP 是一项指标，通过观察用户访问网页期间发生的所有点击、点按和键盘互动的延迟时间，评估网页对用户互动的总体响应情况。最终 INP 值是观测到的最长互动时间，离群值会被忽略。

**根据 INP 的用途，系统仅会观察以下互动类型：**

1. 使用鼠标点击。
2. 在带触摸屏的设备上点按。
3. 按实体键盘或屏幕键盘上的按键。

### 优化方案

1. 优化耗时较长的任务

- 在面向用户的关键任务中让给主线程。
- 使用 postTask() 确定任务的优先级。
- 不妨考虑使用 scheduler.yield() 进行实验。
- 尽量减少在函数中执行的工作。

2. 当屏幕上发生视觉变化时，使用 requestAnimationFrame 来执行函数，而不要使用 setTimeout 或 setInterval，因为后者无法控制其执行时机，这样就会出现跳帧卡顿的问题，前者则可以保证执行时机一直在帧的开头

参考内容：[https://web.dev/articles/inp?hl=zh-cn](https://web.dev/articles/inp?hl=zh-cn)
