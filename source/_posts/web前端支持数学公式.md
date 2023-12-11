---
title: web前端支持数学公式
date: 2023-12-11 20:01:26
tags:
---

## 前言
项目中要引入第三方题库，而题库中存在公式类题目，因此前端要做题目的展示。

## 正文
因为第三方题库是采用MathJax渲染，且已经经过业务的锤炼，因此我们也照猫画虎同样采用MathJax来渲染。

### 1.MathJax是什么
- 官网是这样说明的：MathJax 是一个开源 JavaScript 显示引擎，适用于 LaTeX、MathML 和 AsciiMath 表示法，适用于所有现代浏览器，内置对屏幕阅读器等辅助技术的支持。

- 我们这边用到的是LaTex，因此我这里简单说明一下什么是LaTex：LaTex的基础是Tex，Tex是一个排版系统，是一个拥有自己完整体系的语法，可以用于各种科研、试卷甚至音乐需要，而LaTex建立在Tex的基础上进行性能扩展，相当于Tex的一个宏包。而对于使用者来说，由于LaTex的使用还是有一定的难度和复杂度，大家还是只在公式方面比较青睐它，所以大多数我们常看见的应用也都是与公式相关。

详细细节大家可以自行百度～

### 2.为什么要用MathJax
看到这里大家应该也知道我们为什么要使用MathJax，这里总结一下，两方面原因：

  1. 第三方题库采用了MathJax，切经过了业务的锤炼

  2. MathJax具有非常优秀的兼容性

### 3. 如何使用MathJax
关于这个问题官方文档中写的很详细。这里总结一下：

1. 配置与加载
这里要注意一下版本，v2和v3的版本使用还是有很大差别的，我们项目中用到的是v3的版本，也建议大家使用。html中添加如下代码即可：

```javascript
<script>
  MathJax = {
    tex: {
      inlineMath: [['$', '$'], ['\\(', '\\)']], // 解析行内公式
      displayMath: [["$$", "$$"], ["\\[", "\\]"]],   //段内公式选择符
    },
    svg: {
      fontCache: 'global'
    }
  };
</script>
<script type="text/javascript" id="MathJax-script" async
  src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-svg.js">
</script>
```

大致原理为，MathJax会按照配置找到需要渲染的公式，例如`$$x = {-b \pm \sqrt{b^2-4ac} \over 2a}.$$`转换为

$$x = {-b \pm \sqrt{b^2-4ac} \over 2a}.$$
​
但是MathJax不会单独渲染某个公式，而是整个页面进行渲染，所以牵扯到异步的话可能会有性能问题，关于这点我们后面详细说明。

2. 如何将代码托管在本地
如上我们可以看到，我们引用的是cdn的js，实际项目中肯定不允许，那么我们如何将代码托管在本地呢？
执行npm install mathjax@3或者从github将代码下载下来并当到本地服务器上，相应的js引用路径修改正确就可以了。

3. 最终的配置文件
因为项目中存在多个入口，又不想每个html都加一遍配置代码，所以抽离了一层加载的js，内容如下：

```javascript
/*
 * @Description: mathjax加载方法
 * @Usage: 页面中添加<script type="text/javascript" id="MathJax-script" defer src="/assets/lib/mathjax/load-mathjax-v1.js"></script>
 */
(function () {
  if (!window.MathJax) {
    // amsmath 包是一个 LaTeX 必须掌握的宏集
    // 而\boldsymbol的渲染效果是加粗的斜体字符。 mathbf{a}为正体
    // mhchem 化学公式宏包
    // AMSsymbols各种命令符号
    // 这个 extpfeil 扩展添加了更多用于生成可扩展箭头的宏，包括 \xtwoheadrightarrow ， \xtwoheadleftarrow ， \xmapsto ， \xlongequal ， \xtofrom ，以及非标准 \Newextarrow 创建自己的可扩展箭头

    window.MathJax = {
      tex: {
        inlineMath: [["$", "$"], ["\\(", "\\)"]],   //行内公式选择符
        displayMath: [["$$", "$$"], ["\\[", "\\]"]],   //段内公式选择符
        packages: ['base', 'ams', 'mhchem', 'textmacros', 'color', 'extpfeil', 'amscd']
      },
      options: {
        enableMenu: false, // 隐藏右键菜单
        skipHtmlTags: ["script", "noscript", "style", "textarea", "pre", "code", "a"],   //避开某些标签
        ignoreHtmlClass: "comment-content",
        processHtmlClass: 'tex2jax_process'
      },
      loader: {
        load: ['[tex]/mhchem', '[tex]/textmacros', '[tex]/color', '[tex]/extpfeil', '[tex]/amscd']
      }
    }
  }
  var script = document.createElement('script');
  script.src = '/assets/polyfill/polyfill.min.js';
  document.head.appendChild(script);
  var script = document.createElement('script');
  script.src = '/assets/lib/mathjax/es5/tex-svg.js';
  document.head.appendChild(script);
})();
```

4. 如何处理异步数据
上面我们提到MathJax不会单独渲染某个公式，而是整个页面进行渲染。我们项目中存在许多的公式，不可能有一个公式页面就重新渲染一遍，那么我们如何处理呢？

我们可以在全局加一个防抖，如果100ms内再次执行渲染，那么重新开始计时，直到超过100ms，例如我们vue项目中是这么用的：
```javascript
mathJaxTexReset:
// 使用的lodash的防抖方法
 _.debounce(function(){
 //渲染函数
   this.$utils.mathJaxTexReset()
 }, 100)
```

## 后记
这里只是写一个MathJax的简单使用教程，还有更丰富的内容带大家发掘～

感谢观看～