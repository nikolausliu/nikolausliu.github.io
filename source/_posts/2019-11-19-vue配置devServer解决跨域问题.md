---
title: vue配置devServer解决跨域问题
date: 2019-11-19 10:11:23
tags:
  - vue
  - devServer
  - 跨域
categories: 编程
comments:
---

以前请求的接口都是后端配好跨域的请求头，最近请求某个接口时突然报跨域的错，后端一顿操作无果，行吧，我记得以前看到过 vue 可以通过配置 devServer 解决跨域问题，就配一下吧。

<!-- more -->

先看一种简单的情景，假如要请求`http://xxx.com/api/list`这么一个接口，如果后端没有配置跨域请求头，那在本地开发阶段请求这个接口就是跨域的

```javascript
// 跨域
axios('http://xxx.com/api/list')
```

通过配置 devServer 可以解决跨域问题。

devServer 的原理就是在本地跑了一个 node 服务。前端请求的是本地的 node 服务接口，本地 node 服务端再去请求目标服务器接口，拿到返回结果后，再转发给前端。这里的关键点是：**前端请求本地 node 服务接口，由于是同源的，就不存在跨域问题。本地 node 服务请求目标服务器属于服务端之间的通信，也不存在跨域问题。通过这种转发的方式，跨域问题就解决了**。

看一下配置：

```javascript
// vue.config.js
module.exports = {
  devServer: {
    proxy: {
      '/test': {
        target: 'http://xxx.com',
        changeOrigin: true,
        pathRewrite: {
          '^/test': '',
        },
      },
    },
  },
}
```

上面代码的意思是，遇到接口地址里有`/test`的（这里可以自己定义规则，替换成其它的），代理到`target`服务器。`changeOrigin`是跨域，设为`true`开启就行。`pathRewrite`是路径重写，也就是把请求接口里`/test`替换成空字符串

用了上面的配置以后，请求的时候，直接请求本地接口，并且本地接口要加上匹配规则`/test`。比如，如果项目是运行在 8080 端口下的，那请求的时候就这样写：

```javascript
// 跨域
// axios('http://xxx.com/api/list')
// ->
axios('http://localhost:8080/test/api/list')
```

打开浏览器 Network 面板，会发现实际请求的地址是`http://localhost:8080/api/list`。这正如上面对`pathRewrite`的解释：`/test`被替换成了空字符串。而跨域问题也解决了。

上面`axios('http://localhost:8080/test/api/list')`的这种写法是为了说明问题做了简化，实际项目自然不能这样写，应该配置 axios 的`baseURL`，并根据开发环境和生产环境做个判断：

```javascript
const isDev = process.env.NODE_ENV !== 'production'
axios.defaults.baseURL = isDev ? '/test' : ''

axios('/api/list')
```

这样，开发环境请求的时候就会请求`/test/api/list`，由于是相对路径，就相当于请求了`http://localhost:8080/test/api/list`。而生产环境就会请求`http://xxx.com/api/list`
