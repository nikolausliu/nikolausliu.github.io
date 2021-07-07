---
title: 强大的Array.prototype.reduce
date: 2021-07-07 11:18:39
tags:
  - JavaScript
  - Array
categories:
comments:
---

[Array.prototype.reduce](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce)方法是数组原型方法中比较强大的一个，在某些场景下，用其它方法可能要几步才能完成的，reduce甚至只用一行代码就能搞定。

<!-- more -->

举个最简单的例子，要求数组`[1, 2, 3, 4, 5]`每一项的和，不用reduce的写法：

```javascript
const arr = [1, 2, 3, 4, 5]
const sum = (arr) => {  
  let res = 0
  arr.forEach(item => {
    res += item
  })
  return res
}
sum(arr)  // 15
```

使用reduce来写：

```javascript
const arr = [1, 2, 3, 4, 5]
const sum = (arr) => arr.reduce((acc, cur) => acc + cur)
sum(arr)  // 15
```

可以看到函数体精简到了一行，我们甚至不需要定义中间的暂时变量。下面简单说下用法。

# 用法

```javascript
arr.reduce(callback(acc, cur[, idx[, src]]) [, initialValue])
```

- 该方法接收两个参数：回调函数callback和初始值initialValue，第二个参数是可选的。
- 回调函数callback接收4个参数：
  - 累加器acc
  - 数组迭代过程中的当前项cur
  - 数组迭代过程中的当前索引idx
  - 源数组

回调函数的后三个参数比较好理解，就和数组的`forEach`、`map`等方法的callback参数一致，重点是要理解累加器`acc`和初始值`initialValue`。

调用`reduce`时会遍历数组并调用`callback`，**每一轮遍历`callback`计算出来的返回值就作为一下轮遍历的累加器**。

第一轮遍历和最后一轮遍历比较特殊，需要特别关注。最后一轮遍历的返回结果自然就作为`reduce()`的返回值了。

而第一轮遍历的累加器从哪来呢？这就要说到初始值了，如果传了初始值，那初始值就作为第一轮遍历的累加器，并且数组是从索引0开始遍历的(此时`acc === inititalValue;cur === src[0]`)。如果没传初始值，那数组的第0项就是第一轮遍历的累加器，同时数组是从索引1开始遍历的(此时`acc === src[0];cur === src[1]`)。**所以有初始值会比无初始值多一轮遍历**。

用一个表格来列出每一轮遍历各个值是什么可以帮助我们更清晰地认识这个方法地执行过程，以上面数组求和地方法为例：

| acc | cur | idx | src | returnValue |
| :-: | :-: | :-: | :-: | :-: |
| 1 | 2 | 1 | [1,2,3,4,5] | 3 |
| 3 | 3 | 2 | [1,2,3,4,5] | 6 |
| 6 | 4 | 3 | [1,2,3,4,5] | 10 |
| 10 | 5 | 4 | [1,2,3,4,5] | 15 |

如果给上面的代码提供一个初始值：

```javascript
const sum = (arr) => arr.reduce((acc, cur) => acc + cur, 10)
```

| acc | cur | idx | src | returnValue |
| :-: | :-: | :-: | :-: | :-: |
| 10 | 1 | 0 | [1,2,3,4,5] | 11 |
| 11 | 2 | 1 | [1,2,3,4,5] | 13 |
| 13 | 3 | 2 | [1,2,3,4,5] | 16 |
| 16 | 4 | 3 | [1,2,3,4,5] | 20 |
| 20 | 5 | 4 | [1,2,3,4,5] | 25 |

# 使用场景

了解了用法，我们来看看哪些场景下可以使用`reduce`

## 用reduce实现数组原型的其它方法

上面提到过，`reduce`的`callback`的后3个参数和`forEach`、`map`的`callback`参数一致，他们是如此的相似，实际上`forEach`、`map`等这些数组方法都可以用`reduce`来实现：

```javascript
// 用reduce实现forEach
if (!Array.prototype.reduceForEach) {
  Array.prototype.reduceForEach = function(callback, thisArg) {
    return this.reduce(function(acc, cur, idx, src) {
      callback.call(thisArg, cur, idx, src)
    }, [])
  }
}
```

```javascript
// 用reduce实现map
if (!Array.prototype.reduceMap) {
  Array.prototype.reduceMap = function(callback, thisArg) {
    return this.reduce(function(acc, cur, idx, src) {
      acc[idx] = callback.call(thisArg, cur, idx, src)
      return acc
    }, [])
  }
}
```

其它方法也是类似的思路，就不展开了。实现的过程需要注意：**必须提供初始值[]**，因为不提供初始值遍历会从索引1开始。另外要注意**要实现的方法的返回值和是否需要改变原数组**。

## 数组去重

```javascript
const arr = [1, 1, 2, 2, 3, 3]
const unique = function(arr) {
  return arr.reduce((acc, cur) => {
    if (!acc.includes(cur)) {
      acc.push(cur)
    }
    return acc
  }, [])
}
unique(arr) // [1, 2, 3]
```

## 二维数组转一维

```javascript
[[1, 2], [3, 4], [5, 6]].reduce((acc, cur) => acc.concat(cur), [])  // [1, 2, 3, 4, 5, 6]
```

## 计算最小值最大值

```javascript
[{value: 1}, {value: 2}, {value: 3}].map(el => el.value).reduce((acc, cur) => Math.max(acc, cur), -Infinity)
```
这里有两点需要注意：

1. 基本思路是用累加器和每一项比，用`Math.max`求出每一轮遍历的最大值，最后一轮算出的返回值就是最大值。同时因为要算的是最大值，所以初始值是`-Infinity`
2. 中间要用`map`转一下是为了保证累加器在每一轮计算出的结果的类型保持一致。

## 计算数组中每个元素出现的次数

```javascript
[1,2,3,4,1,3,1].reduce((acc, cur) => {
  if (!(cur in acc)) {
    acc[cur] = 1
  } else {
    acc[cur] += 1
  }
  return acc
}, {})  // {1: 3, 2: 1, 3: 2, 4: 1}
```

## 斐波那契数列

斐波那契数列由`0`和`1`开始，后面的每一项数字都是前面两项数字的和，形如`0,1,1,2,3,5,8,13...`。这也是[leetcode上的第509题](https://leetcode-cn.com/problems/fibonacci-number/)，不过它的题目内容是求第N项是多少，比如`F(0) === 0`，`F(1) === 1`,`F(2) === 1`。

如果是求前N项斐波那契数列组成的数组用`reduce`可以很容易的实现：

```javascript
function fibonacci(n) {
    if (n === 0) return [0]
    if (n === 1) return [0, 1]
    const arr = Array(n - 1).fill(1)
    return arr.reduce((acc) => {
        acc.push(acc[acc.length - 1] + acc[acc.length - 2])
        return acc
    }, [0, 1])
}
fibonacci(0)  // [0]
fibonacci(1)  // [0,1]
fibonacci(2)  // [0,1,1]
fibonacci(3)  // [0,1,1,2]
fibonacci(4)  // [0,1,1,2,3]
```

这里有以下几个关键点：

1. 第0项和第1项是固定的直接返回
2. 从第2项开始使用`reduce`，并且初始值是第1项的`[0, 1]`
3. 解释下`const arr = Array(n g- 1).fill(1)`，由于前两项已经固定了，所以使用`reduce`需要构造一个剩下项数位的数组，为什么是`n - 1`，自己在心里用临界值推算下就知道了。而为什么要`fill(1)`呢？这就是个小小的知识点了，因为`Array(n)`构造出来的数组虽然`length`是`n`，但是每一项都是空（在chrome打印出来是`[empty x n]`）是不参与遍历的，必须填充一个实际的值才能遍历这个数组，填充什么值是无所谓的。
4. `reduce`函数体的内容用`n === 2`推算下就知道了。当`n === 2`时，累加器是`[0, 1]`，现在要做的是，向累加器内`push 1`，这个`1`怎么得到的呢？就是前一项的`[0, 1]`的最后两项的和即`acc[acc.length - 1] + acc[acc.length - 2]`。

# 总结

`reduce`的用法远不止于此，用好了可以大大的简化代码。我总结了下，发现当你你要做的事是**要遍历一个数组，并且上一轮的遍历的结果会传递到下一轮遍历的过程中去的时候**，基本都能使用`reduce`来解决，和递归的思想有点像，又好像不完全一样。
