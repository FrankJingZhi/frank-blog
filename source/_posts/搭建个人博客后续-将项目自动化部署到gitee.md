---
title: 搭建个人博客后续-将项目自动化部署到gitee
date: 2023-12-22 14:17:32
tags: [GitHub Actions, Gitee Pages, 自动化]
---

## 前言

之前的文章——[搭建个人博客](/frank-blog/2023/12/13/搭建个人博客)中说到还欠缺一个部署到gitee pages的功能，现在已经实现了，今天来还愿。

## 正文

### 第一次部署

只需要在gitee pages页面选择部署分支（这个就跟github一致就可以了）、部署目录（不用填）、强制使用HTTPS（是），点击确定，等待几分钟即可。打开[我们的网址](https://frank-awesome.gitee.io/frank-blog)就可以看到我们部署的内容啦！

### 后续部署

后续我们在提交新内容并部署时，发现新内容并不会自动部署到`gitee pages`。原因是我们的`CI`代码中只是将`github仓库`中的代码同步到`gitee仓库`，并没有将`gitee仓库`中的代码部署到`gitee pages`。

有的同学说：“那既然没有添加部署能力，那我们像github一样添加一下不就得了？”答案是：不可以。gitee的免费版并没有提供像github actions的编译部署能力，对应的能力是在gitee pages pro版本中，是需要花钱的。那咋办呢？

既然这样我们就看看能不能直接用github actions的能力来处理。我们在[官方市场](https://github.com/marketplace?type=actions)中通过关键词`gitee pages`搜索，找到如下结果：
![](/images/gitee-pages-search-result.png)

我们选择一个stars数量多的`Gitee Pages Action`，看下描述：
> 为了实现 Gitee Pages 的自动部署，我开发了 Gitee Pages Action ，只需要在 GitHub 项目的 Settings 页面下配置 keys，然后在 .github/workflows/ 下创建一个工作流，引入一些配置参数即可。

完全满足我们的需求，下面再看下示例：

```yml
name: Sync

on:
  push:
    branches: [main]
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Sync to Gitee
        uses: wearerequired/git-mirror-action@master
        env:
          # 注意在 Settings->Secrets 配置 GITEE_RSA_PRIVATE_KEY
          SSH_PRIVATE_KEY: ${{ secrets.GITEE_RSA_PRIVATE_KEY }}
        with:
          # 注意替换为你的 GitHub 源仓库地址
          source-repo: git@github.com:doocs/leetcode.git
          # 注意替换为你的 Gitee 目标仓库地址
          destination-repo: git@gitee.com:Doocs/leetcode.git

      - name: Build Gitee Pages
        uses: yanglbme/gitee-pages-action@main
        with:
          # 注意替换为你的 Gitee 用户名
          gitee-username: yanglbme
          # 注意在 Settings->Secrets 配置 GITEE_PASSWORD
          gitee-password: ${{ secrets.GITEE_PASSWORD }}
          # 注意替换为你的 Gitee 仓库，仓库名严格区分大小写，请准确填写，否则会出错
          gitee-repo: doocs/leetcode
          # 要部署的分支，默认是 master，若是其他分支，则需要指定（指定的分支必须存在）
          branch: main
```

其中`Sync to Gitee`同步到gitee这一步我们之前已经实现了，可以忽略。只需要关注`Build Gitee Pages`这一步就可以了，我们按照要求填写即可。注意：`gitee-password: ${{ secrets.GITEE_PASSWORD }}`中`GITEE_PASSWORD`是要在`github -> settings -> secrets`中配置，配置完成后如下：

```yml
# 同步github项目代码到gitee.
mirroring-gitee:
  needs: [build, deploy]
  runs-on: ubuntu-latest
  steps:
    # nodejs使用的版本
    - name: Use Node.js 16.x
      uses: actions/setup-node@v2
      with:
        node-version: '16'
    - name: Mirror the Github organization repos to Gitee.
      uses: Yikun/hub-mirror-action@master
      with:
        src: 'github/FrankJingZhi'
        dst: 'gitee/Frank-Awesome'
        dst_key: ${{ secrets.GITEE_PRIVATE_KEY }}
        dst_token: ${{ secrets.GITEE_TOKEN }}
        static_list: "frank-blog"
        force_update: true
        timeout: '5m'
    # 把项目部署到gitee pages
    - name: Build Gitee Pages
      uses: yanglbme/gitee-pages-action@main
      with:
        gitee-username: Frank-Awesome
        gitee-password: ${{ secrets.GITEE_PASSWORD }}
        gitee-repo: Frank-Awesome/frank-blog
        branch: main
```

再次运行：
![](/images/gitee-pages-action-result.png)

打开我们的[gitee pages页面](https://frank-awesome.gitee.io/)看下：
![](/images/gitee-pages-page.png)


好了，现在我们就实现了gitee pages的自动部署功能✨

## 后序

完结撒花🎉

如果有错误或者不严谨的地方，请务必给予指正，十分感谢。如果喜欢或者有所启发，欢迎评论、star，对作者也是一种鼓励。