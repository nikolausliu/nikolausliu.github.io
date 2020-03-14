---
title: 浅谈vue指令v-model
date: 2019-02-28 13:34:28
tags:
  - vue
categories: 编程
comments:
---

`v-model`实际是一种语法糖

<!-- more -->

```html
<input v-model="text" />
```

上面的写法等价于:

```html
<input :value="text" @input="text = $event.target.value" />
```

知道这一点在开发中是很重要的。比如 ElementUI 的 switch 组件，如果按照官网的写法用`v-model`绑定状态，那么当你点击 switch 组件后组件状态会立马变更。而实际开发 switch 组件往往是需要点击后请求接口，拿到接口 response 后再异步改变组件状态的。利用`v-model`的语法糖原理，可以轻松的做到异步改变状态：

```vue
<template>
  <el-switch :value="status" @input="handleInput($event)"></el-switch>
</template>

<script>
export default {
  data() {
    return {
      status: true,
    }
  },
  methods: {
    input(checked) {
      // 模拟异步
      setTimeou(() => {
        this.value = checked
      })
    },
  },
}
</script>
```
