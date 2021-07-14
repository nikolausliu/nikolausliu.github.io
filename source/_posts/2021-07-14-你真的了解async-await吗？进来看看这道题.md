---
title: 你真的了解async...await吗？进来看看这道题
date: 2021-07-14 17:45:19
tags:
  - JavaScript
categories:
comments:
---

偶然间看到下面这道题，是考察`async...await`机制的，我觉得还挺有意思的。你可以试试不借助控制台自己在心里推算一下运行结果。

```javascript
const Err = async () => {
	throw new Error(42);
};

const Obj = {
	async A (){
		try {
			return Err();
		} catch {
			console.log('A');
		}
	},
	async B (){
		try {
			await Err();
		} catch {
			console.log('B');
		}
	},
	async C (){
		try {
			Err();
		} catch {
			console.log('C');
		}
	},
};

( async () => {
	for( const key in Obj )
	{
		try {
			await Obj[key]();
		} catch {
			console.log('D');
		}
	}
} )();
```

<!-- more -->

放一张图防止你们不小心看到答案。

![](/images/common/feedpig.jpg)

先说答案，分别打印`D`和`B`，最后报错`Uncaught (in promise) Error: 42`

我总结了一下，这题主要考察了以下知识点：

# `async`函数总是返回一个promise

这个promise是什么，分为以下情况：

1. 函数体内抛出了错误，则函数执行结果为这个错误的`Promise.reject()`包装对象
2. 如果函数体内返回了一个promise，则函数执行结果就是这个promise。
3. 否则，函数执行结果为函数体内返回值的`Promise.resolve()`包装对象。有一种情况容易让人混淆：函数体`return new Erro(1)`，则执行结果相当于`Promise.resolve(new Error(1))`，注意`return new Error(1)`和`throw new Error(1)`的区别。
    
```
async function fn1() {
    throw new Error(1)
}
async function fn2() {
    return Promise.reject(1)
}
async function fn3() {}
async function fn4() {
    return new Error(1)
}
fn1()   // Promise {<rejected>: Error: 1}
fn2()   // Promise {<rejected>: 1}
fn3()   // Promise {<fulfilled>: undefined}
fn4()   // Promise {<fulfilled>: Error: 1}
```

# `await`在等待解开一个promise，等不来就报错

上面的标题是为了简短一点，可能描述地不是很准确，我会拆分成几点来说明：

1. `await`后总是跟一个promise，如果后面跟的不是promise，会用`Promise.resolve()`包装一下。
2. 如果`await`表达式后跟一个`fulfilled`态的promise，会返回对应的值。
3. 如果`await`表达式后跟一个`rejected`态的promise，会抛出错误，可以用`.catch()方法`或`try...catch`块捕获。

```javascript
async function fn1() {
    const res = await Promise.resolve(1)
    console.log(res)
}
async function fn2() {
    const res = await new Error(1) // 相当于Promise.resolve(new Error(1))
    console.log(res)
}
async function fn3() {
    const res = await Promise.reject(1)
    console.log(res)    
}
fn1()   // 1
fn2()   // Error: 1
fn3()   // 报错 Uncaught (in promise) 1
// fn3改写成下面两种形式都能捕获错误
async function fn3() {
    const res = await Promise.reject(1).catch(e => {
        console.log(e)
    })
}
async function fn3() {
    try {
        const res = await Promise.reject(1)   
    } catch(e) {
        console.log(e)
    }
}
```

了解了上面的知识点，我们就可以分析代码了，我把代码解释放在了下面的代码注释中，这样看起来应该更清晰一点：


```javascript
const Err = async () => {
	throw new Error(42);
	// 相当于 return Promise.reject(new Error(42))
};

const Obj = {
	async A (){
	    // 1.2 for循环中`await A()`相当于`await Promise.reject(new Error(42))`
	    // 会被for循环内的catch块捕获
	    // **所以第1轮for循环输出D**
		try {
			return Err();
			// 1.1 相当于 return Promise.reject(new Error(42))
			// 这行代码没有抛出错误，所以下面的catch块不会走，不会输出A
		} catch {
			console.log('A');
		}
	},
	async B (){
	    // 2.2 for循环中`await B()`相当于`await Promise.resolve(undefined)`
	    // 这行代码不会被for循环内的catch块捕获
	    // **所以第2轮for循环输出B**
		try {
		    // 2.1 相当于`await Promise.reject(new Error(42))`
		    // 这行代码会被下面的catch块捕获，输出B
			await Err();
		} catch {
			console.log('B');
		}
	},
	async C (){
	    // 3.2 for循环中`await C()`相当于`await Promise.resolve(undefined)`
	    // 不会被for循环内的catch块捕获
	    // **所以第3轮什么都不输出。但是会报错`Uncaught (in promise) Error: 42`**
		try {
		    // 3.1 相当于`Promise.reject(new Error(42))`
		    // 这行代码不会被下面的catch块捕获，不会输出C
		    // 但是由于这个`rejected`态的promise没有被catch处理所以会报错`Uncaught (in promise) Error: 42`
		    // 注意try...catch块内`Promise.reject()`和`await Promise.reject()`是不一样的。
			Err();
		} catch {
			console.log('C');
		}
	},
};

( async () => {
	for( const key in Obj )
	{
		try {
			await Obj[key]();
		} catch {
			console.log('D');
		}
	}
} )();
```


