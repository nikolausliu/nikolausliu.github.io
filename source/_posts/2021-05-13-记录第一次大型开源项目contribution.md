---
title: 记录第一次大型开源项目contribution
date: 2021-05-13 17:42:18
tags: 
  - github
categories: 编程
comments:
---

![](https://nuxtjs.org/logos/nuxtjs-typo.svg)

题目起得有点唬人，有点标题党了。实际上就只是改了[nuxtjs官方文档的一处谬误](https://nuxtjs.org/docs/2.x/directory-structure/pages#the-watchquery-property)。这里是[PR地址](https://github.com/nuxt/nuxtjs.org/pull/1431)。

<!-- more -->

现在来简单解释一下这个谬误：

浏览器地址栏`query`发生改变后，nuxtjs出于性能考虑默认是不会重新调用`asyncData, fetch`等来重新拉取数据的。通过`watchQuery`选项你可以自定义这个行为。但是问题出在nuxtjs在2.12版本后推出的`new fetch hook`是不受`watchQuery`影响的，我上面贴出的那一页官方文档却没有把这个问题交代清楚，它原本的话是这样的：

> Use the `watchQuery` key to set up a watcher for query strings. If the defined strings change, all component methods (asyncData, **fetch**, validate, layout, ...) will be called. Watching is disabled by default to improve performance.

并没有体现出新的`fetch`不受影响这一点，所以我把`fetch`改成了`fetch(context)`（新老`fetch`的一个区别就是老的有`context`参数，新的没有）。

另外把下面alert提示信息也改了：

老的：

> If you want to set up a watcher for all query strings, set `watchQuery` to `true`

新的：

> **Warning**: The new `fetch` hook introduced in 2.12 is not affected by `watchQuery`. For more information see [listening to query string changes](/docs/2.x/features/data-fetching#the-fetch-hook).

贴一下文档截图和PR截图

![官方文档修改](http://ww1.sinaimg.cn/large/d7f38664ly1gqgy26197kj20s00lxgo5.jpg)

![Pull request](http://ww1.sinaimg.cn/large/d7f38664ly1gqgy25xus5j20v60q2jtx.jpg)

