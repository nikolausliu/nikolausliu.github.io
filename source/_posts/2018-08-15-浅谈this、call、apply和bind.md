---
title: 浅谈this、call、apply和bind
date: 2018-08-15 21:24:56
tags: 
  - JavaScript
  - this
categories: 
comments:
---
this在javascript中是一个比较重要的概念，今天来谈一谈this相关的知识。
<!-- more -->

# this
## this指向

this的指向，记住这句话：**谁调用了this，this就指向谁**。

```javascript
var a = 1
var o = {
  a: 2,
  fn: function(){
    console.log(this.a)
  }
}
// (1)
o.fn()  // 2
// (2)
var fn = o.fn
fn()  // 1
// (3)
o.fn.call({a: 3}) // 3
```
上面的代码，（1）o.fn() 由于是 o 调用了 fn ，所以 this 指向 o ，输出 o.a 即2。（2）这里先是定义了一个匿名函数，匿名函数引用o.fn，最终执行时是**匿名函数自执行**，对于这种情况，this在非严格模式下指向window，在严格模式下指向undefined。我们知道全局变量是作为属性挂在window下的，所以这里输出1。（3）这里用call函数改变了this的指向，输出3。与call相似的函数还有apply和bind,它们都可以改变this的指向。

## 箭头函数里的this指向

关于箭头函数的this指向，有很多的讨论：

- 廖雪峰：**箭头函数内部的this是词法作用域，由上下文确定。**
- 阮一峰：**函数体内的this对象，就是定义时所在的对象，而不是使用时所在的对象。**、**箭头函数里面根本没有自己的this，而是引用外层的this**。

```javascript
var a = 1
var o = {
  a: 2,
  fn: () => {
    console.log(this.a)
  },
  fn2: function() {
    // console.log(this.a) // fn2不是箭头函数，调用o.fn2()时它的this指向o。foo是箭头函数，它的this指向fn2,所以也指向o
    var foo = () => {
      console.log(this.a)
    }
    foo()
  },
  fn3: ()=>{
    var foo = () => {
      console.log(this.a)
    }
    foo()
  }
}
// (1)
o.fn()  // 1
// (2)
o.fn2() // 2
// (3)
o.fn3() // 3
```

抛开词法作用域那些概念，我们分析一下这句话**箭头函数里面根本没有自己的this，而是引用外层的this**。可以分成两种情况：(1)箭头函数作为一个对象的属性存在，此时箭头函数内this指向对象所在上下文。比如上述代码中，fn作为对象o的属性存在，所以fn的this指向对象o所在上下文window。（2）如果不是第一种情况，那么箭头函数内this指向包裹箭头的环境的this。比如上面代码中fn2包裹了箭头函数foo,所以foo的this指向fn2的this。如果是箭头函数套箭头函数，会一层一层向外指。比如上面fn3套foo。那么foo没有自己的this，所以指向fn3,fn3也没有自己的this,所以指向对象o的上下文window。

我们知道setTimeout里的this总是指向window，所以在setTimeout里我们想使用setTimeout所处上下文的this时，我们需要写类似于`var _this = this`这样的代码来把当前的this暂存起来，箭头函数可以让我们避免这样的工作：

```javascript
// ES5
var a = 1
var o = {
  a: 2,
  fn: function(){
    var _this = this
    setTimeout(function(){
      console.log(this.a)   // 1
      console.log(_this.a)  // 2
    })
  }
}

// ES6
var a = 1
var o = {
  a: 2,
  fn: function(){
    setTimeout(() => {
      console.log(this.a) // 2
    })
  }
}
```

由于箭头函数的this在定义的时候已经确定了，所以无法使用call、apply和bind这些方法了。对箭头函数使用这些方法，传入这些方法的第一个参数（指定的this）会被忽略，当然其它的参数还是会被传入箭头函数执行。

```javascript
var foo = a => {
  console.log(this, a)
}
foo(1)  // window 1
foo.call([1,2,3], 1)  // window 1
```


# call和apply
call和apply是定义在Function.prototype上的原型方法，所有的函数都通过原型链继承了这两个方法，可以直接在函数上调用，使用时就像这样`fn.call(thisArg, arg1, arg2, ...)`。

call和apply的第一个参数都是将要绑定的this值。在非严格模式下，如果第一个参数指定为undefined或null，this会被绑定为window；如果第一个参数被指定为字符串、数字或布尔值，this会被绑定为相应的包装对象，比如`new Number(1)`。

主要区别在于第一个参数后面的参数。call接收多个参数，而apply接受多个参数组成的数组或类数组对象。

```javascript
var foo = {
  name: 'foo',
  fn: function(a, b) {
    console.log(this.name, a, b)
  }
}
var fn = foo.fn
fn.call({name: 'bar'}, 1, 2)    // bar 1 2
fn.apply({name: 'bar'}, [1, 2]) // bar 1 2
```

以上代码可以看到，通过call和apply把fn的this指向由foo指向了`{name: 'bar'}`，区别在于call分别传了1和2两个参数分别对应fn的参数a和b，而apply把1和2组成了一个数组传入。使用call和apply时，可以根据自己的实际需求，哪个方便用哪个。


# bind

bind和call、apply类似，也是用来绑定this的，bind的参数和call是一致的，区别在于：call和apply调用时会直接调用原函数，只是为其指定了新的this。但是调用bind并不会直接执行原函数，而是会返回一个新的函数，这个函数的this是绑定之后的，你可以在这之后随时调用这个绑定之后的新函数。

```javascript
var foo = {
  name: 'foo',
  fn: function(a, b){
    console.log(this.name, a, b)
  }
}
var fn = foo.fn
var boundFn = fn.bind({name: 'bar'}, 1, 2)
boundFn() // bar 1 2
boundFn() // bar 1 2
```

上面的代码，先把对象foo的fn方法保存在变量fn上，然后对fn调用bind，返回了一个绑定了this值和初始参数的绑定函数，之后可以方便的随时调用这个绑定函数。但是这种方便也只是部分范围内的方便，如果待绑定函数是有参数的，那么调用bind时为其传入参数，返回的新函数也讲被绑定上了初始参数。就比如上例中的boundFn是绑定了初始参数1,2，如果想要向fn传入参数3,4，我需要再获取一个新的绑定函数实例`var boundFn2 = fn.bind({name: 'bar'}, 3, 4)`。

使用bind绑定过this的函数，对其使用call和apply，也无法再改变其this指向了，其行为和对箭头函数调用call和apply类似：第一个参数会被忽略。

```javascript
var foo = {
  name: 'foo',
  fn: function(a, b){
    console.log(this.name, a, b)
  }
}
var fn = foo.fn
var boundFn = fn.bind({name: 'bar'}, 1, 2)
boundFn() // bar 1 2
boundFn() // bar 1 2

// 对绑定函数调用call,第一个参数会被忽略
boundFn.call({name: 'foo bar'}, 1, 2) // bar 1 2  
```

## bind的应用
关于bind的应用，在MDN上有一个[小例子](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/bind#Example:_Creating_shortcuts)，我把它摘抄在下面：

> 你可以用 Array.prototype.slice 来将一个类似于数组的对象（array-like object）转换成一个真正的数组，就拿它来举例子吧。你可以简单地这样写：
>
>  ```javascript
>  var slice = Array.prototype.slice;
>
>  // ...
>
>  slice.apply(arguments);
>  ```
> 用 bind()可以使这个过程变得简单。在下面这段代码里面，slice 是 Function.prototype 的 apply() 方法的绑定函数，并且将 Array.prototype 的 slice() 方法作为 this 的值。这意味着我们压根儿用不着上面那个 apply()调用了。
> 
> ```javascript
> // 与前一段代码的 "slice" 效果相同
> var unboundSlice = Array.prototype.slice;
> var slice = Function.prototype.apply.bind(unboundSlice);
> 
> // ...
> 
> slice(arguments);
> ```

这个例子有点绕人，可以先把apply看成一个普通函数，先看bind。我们在对普通函数调用bind时，实际上就是把bind的第一个参数绑定到普通函数的this上，然后返回这个绑定了this的新函数。同理，这里可以理解成`apply.bind`返回了一个**this绑定为unboundSlice的一个新的apply函数**，这个函数也就是`slice`。**apply的this绑定了unboundSlice**这句话可以理解为：apply的执行上下文为unboundSlice，就相当于是unboundSlice调用了apply。所以`slice(arguments)`也就相当于`unboundSlice.apply(arguments)`了。
