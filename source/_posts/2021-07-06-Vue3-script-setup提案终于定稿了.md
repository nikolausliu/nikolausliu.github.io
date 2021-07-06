---
title: Vue3 script setup提案终于定稿了
date: 2021-07-06 20:32:27
tags:
  - vue3
---

vue3的`<script setup>`提案处于实验阶段已经几个月了，当我们用vite的vue模板时终端会提示我们这仍是一个实验性提案，并且建议我们如果使用这个写法要锁vue版本以避免breakage。所以之前我只是对这个提案做了个了解，并没有在工作中使用这个写法。终于我们在2021年6月29日上午迎来了他的[Finalization](https://github.com/vuejs/rfcs/pull/227#issuecomment-870105222)，下面简单梳理下这个定稿的内容。

<!-- more -->

以下内容会在3.1.3版本中被实装，但是要到3.2版本才会正式移除这个提案的“实验性”状态，同时下面提及的将被废弃API也要到3.2版本才会被移除，再此之前新老API将共存一段时间。

# 废弃`useContext`API

之前的用法是这样的：

```
import { useContext} from 'vue'
const { attrs, emit, expose, slots } = useContext()
```

3.2版本后这个API就将被移除，被拆分成几个新的API来取代

# 新增 `useAttrs`API

`attrs`是父组件传递给子组件的属性中除了`props`、`class`以及`style`外的属性：

```
// 父组件
<child a="1" b="2" msg="hello" class="child" style="color:red" />
```

比如上面的代码，父组件向子组件传递了若干属性，我们假设`child`组件定义了一个`msg`prop，那么子组件接收到的`attrs`就是`{a: "1", b: "2"}`。

老的用法：

```
import { useContext} from 'vue'
const { attrs } = useContext()
console.log(attrs)
```

新的用法：

```
import { useAttrs } from 'vue'
const attrs = useAttrs()
console.log(attrs)
```

# 新增 `useSlots`API

`useSlots`一般是用JSX才会用到，做个了解，下面提供新的写法，老的写法就是把`useSlots`换成`const slots = useContext()`就行了：

```
// 父组件
<child>
  <div>默认插槽</div>
  <template #left>
    <div>具名插槽</div>
  </template>
</child>
```

```
// 子组件
import { defineComponent, useSlots } from 'vue'
const Child = defineComponent({
  setup() {
    const slots = useSlots()

    // 渲染组件
    return () => (
      <div>
        // 默认插槽
        <div>{ slots.default ? slots.default() : '' }</div>
        // 具名插槽
        <div>{ slots.left ? slots.left() : '' }</div>
      </div>
    )
  },
})
export default Child
```

# 新增 `defineExpose`API

当使用`<script setup>`时，可以把`<template>`看成是`setup`作用域内的一个function，这样`setup`就相当于一个闭包，除了内部的`<template>`谁都访问不到其作用域内的数据。就像下面的代码这样：

```
function setup() {
  const a = 1
  const b = 2

  return function template() {
    // has access to `b` but doesn't necessarily uses it
    return `<div>${a}</div>`
  }
}
```

所以，传统setup写法里我们可以在父组件中通过子组件的ref实例来访问子组件的数据和方法（`childRef.value.someFn()`），到了`<script setup>`里这一套就行不通了，需要显示地向外暴露你想暴露的内容。

用老的`useContext`API是这么玩得：

```
// 子组件
<script setup>
import { useContext } from 'vue'
const { expose } = useContext()
const a = 1
const b = 2
expose({
    a
})
</script>

// 父组件
<template>
  <child ref="childRef" />
</template>
<script setup>
import { ref } from 'vue'
const childRef = ref()
console.log(childRef.value.a)   // 1
console.log(childRef.value.b)   // undefined
</script>
```

用新的`defineExpose`API:

```
import { defineExpose } from 'vue'
const a = 1
const b = 2
defineExpose({
    a
})
```

# ~~`defineEmit`~~ -> `defineEmits`

`defineEmit`API被更名为`defineEmits`了，因为不管是选项`emits,props`还是API`defineProps`都是复数，之前的`defineEmit`就显得有点扎眼了，统一改成复数了，用法不变。

```
import { defineEmits } from 'vue'
const emit = defineEmits(['change', 'close'])
emit('change', 'change事件的payload')
emit('close', 'close事件的payload')
```

# 新增 `withDefaults`API
当我们在vue中使用TS时，定义props类型有两种方法：

1. 是用原生js的构造函数`String, Boolean, Number, Array`等：

```
import { defineProps } from 'vue'

defineProps({
  name: {
    type: String,
    default: 'Niko',
  },
  age: Number
})
```

2. 用TS类型来定义：

```
import { defineProps } from 'vue'

defineProps<{
  name: string
  age: number
}>()
```

但是，使用第二种方式时，是无法定义props默认值的。使用新增的 `withDefaults`API就能实现props默认值定义，它接收两个参数，第一个是`defineProps()`，第二个是默认值：

```
import { defineProps, withDefaults } from 'vue'

withDefaults(defineProps<{
  name: string
  age: number
}>(), {
  name: 'Niko',
  age: 18
})
```

# 恢复指令的`v`前缀

现在要使用`v-my-dir`这样的指令，在定义指令变量是必须使用`vMyDir`，也就是加上`v`前缀。

```
<script setup>
  import { directive as vClickOutside } from 'v-click-outside'
</script>

<template>
  <div v-click-outside />
</template>
```

# 顶级`await`支持
现在可以直接在`<script setup>`块的顶级写`await`而不用放在`async`函数内了，会被自动编译成`async setup()`。

# 移除`<template>`上的`inherit-attrs`属性

# 如何定义诸如`name`这样的选项？

这一条和上一条一起回答，用一个平级于`<script setup>`的独立的`<script>`块来定义：

```
<script>
  export default {
    name: 'CustomName',
    inheritAttrs: false,
    customOptions: {},
  }
</script>

<script setup>
  // script setup logic
</script>

```

See [Declaring Additional Options](https://github.com/vuejs/rfcs/pull/227#declaring-additional-options) and [Automatic Name Inference](https://github.com/vuejs/rfcs/pull/227#automatic-name-inference).

# 参考

- [https://github.com/vuejs/rfcs/pull/227#issuecomment-870105222](https://github.com/vuejs/rfcs/pull/227#issuecomment-870105222)
- [https://github.com/vuejs/rfcs/blob/master/active-rfcs/0040-script-setup.md](https://github.com/vuejs/rfcs/blob/master/active-rfcs/0040-script-setup.md)

