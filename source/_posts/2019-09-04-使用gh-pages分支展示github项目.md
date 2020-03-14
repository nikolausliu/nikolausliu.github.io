---
title: 使用gh-pages分支展示github项目
date: 2019-09-04 17:42:46
tags:
  - github
  - gh-pages
categories: 编程
comments:
---

很多人都使用 github 的`${username}.github.io`仓库来托管自己的静态博客，但其实 github 的每个仓库都支持通过`${username}.github.io/${repoName}`的方式来访问，只需要创建一个`gh-pages`分支

<!-- more -->

比如，我创建了一个名为`demo`的仓库，在`master`分支下存放 demo 的 vue 源码，如果我想展示这个项目 build 出来后是什么样子，只需要在这个仓库下创建一个`gh-pages`分支，然后把`npm run build`后`dist`文件夹下的文件 push 到这个分支下，就可以使用`http://${username}.github.io/demo`这样的方式来预览这个项目的实际效果了。

但是这样做，每次修改了源码后，还要手动把 build 后的文件 push 到`gh-pages`分支，很不方便。实际上，有开源的工具[gh-pages](https://www.npmjs.com/package/gh-pages)能让我们更方便的完成这个操作。

首先`npm i -D gh-pages`安装这个依赖

然后在`package.json`中添加一条命令:

```json
// package.json
{
  "scripts": {
    "deploy": "gh-pages -d dist"
  }
}
```

这样，每次修改了源码后，只需要先 build 一下，然后`npm run deploy`就自动部署到 gh-pages 分支了

另外，需要注意的一点，如果你的`${username}.github.io`仓库绑定了域名，那么你需要展示的仓库需要添加域名解析的 CNAME 文件。

这里以我的一个用来展示自己封装的 vue 组件的仓库[components-repo](https://github.com/nikolausliu/components-repo)为例，由于我的`nikolausliu.github.io`仓库绑定了`nikolausliu.top`域名，所以我需要添加内容为`nikolausliu.top`的 CNAME 文件。我的这个项目是使用`@vue/cli`搭建的，在根目录下的 public 文件夹下新建一个 CNAME 文件（新建一个 CNAME.txt 文件，里面输入`nikolausliu.top`，然后重命名把扩展名去掉就好了，在 vscode 里可以直接新建不带扩展名的 CNAME 文件），然后 build 后就会自动把 CNAME 文件也打包到 dist 目录了。

还有一点需要注意，就是要正确配置`publicPath`，配置部队的话，css 和 js 等资源将无法正确访问，还是以我上面的项目为例，仓库名为`components-reop`，则 vue.config.js 配置如下：

```javascript
module.exports = {
  outputDir: 'dist',
  publicPath: process.env.NODE_ENV === 'production' ? '/components-repo/' : '/',
}
```

如果还要深挖的话，自动化部署的过程应该可以利用 `github actions` 功能，不过这一块之前只是简单看了下，还没使用过，这里就不做说明了，以后深入学习了`github actions`后再单独写一篇博文
