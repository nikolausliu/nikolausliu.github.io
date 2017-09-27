---
title: 简单封装localStorage的相关操作--store.js
date: 2017-06-29 15:29:51
tags: 
- javascript
- 封装
categories: 编程
comments:
---

# 不是引言的引言

感觉引言好浪费时间，以后尽量不写引言了。

# 正文

localStorage是Html5新增的特性用来提供浏览器的本地存储功能。
<!-- more -->
列举几个localStorage的常用api：

```javascript
// 写、改
localStorage.setItem('user_name', 'Nikolaus');

// 读
localStorage.getItem('user_name');

// 移除某一项
localStorage.removeItem('user_name');

// 清除所有数据
localStorage.clear();
```

掌握了基本的用法，就可以在实际的工作中用到了，比如：你可能希望在做登录成功的时候储存用户名和`token`等标识，就可以这样做:

```javascript
function (){
  // login success code here
  localStorage.setItem('user_name', 'Nikolaus');
  localStorage.setItem('token': 'some token string');
}
```

看起来这样就ok了。但是上面的操作是添加一条数据就要调用一次`localStorage.setItem()`,这一点都不酷。而且，如果你有多条数据需要储存呢？是不是把这些数据包装成一个对象只执行一次储存操作更好呢？而且json格式的数据格式本来就是大部分的场景会遇到的。现在问题来了：localStorage会把你传入的所有值转换为字符串后再保存：

```javascript
localStorage.setItem('a': {'user_name': 'Nikolaus'});
localStorage.setItem('b': 1);
localStorage.setItem('c': true);

localStorage.getItem('a');  // "[object Object]"
localStorage.getItem('b');  // "1"
localStorage.getItem('c');  // "true"
```

莫慌，可以用`JSON.parse`和`JSON.stringify()`在对象和字符串之间互相转换。字符串和布尔类型的值也会被按照预期的转换：

```javascript
JSON.stringify(true); // 'true'
JSON.parse('true'); // true
JSON.stringify(1);  // '1'
JSON.parse('1');  // 1
```

操练一遍：

```javascript
var data = {
  'user_name': 'Nikolaus',
  'token': 'some token string'
};
// 写入
localStorage.setItem('test', JSON.stringify(data));

// 读取
JSON.parse(localStorage.getItem('test'));
```

问题是解决了，可是每次用的时候都要写这么老长一段代码，是不是可以把一些常用的操作封装起来呢？这样以后用到的时候就方便了。实际上，早就有人这么干了，在github上搜索`store.js`可以发现有很多`repo`（为什么叫`store.js`呢？因为store给我的第一印象是商店，用谷歌翻译之后才知道store还指“贮存;（在计算机里）存储”。好吧，英语还是挺重要^_^,那我也就叫它`store.js`吧)。

写之前，先分析一波：我可能需要把不同的数据保存在不同的key上，那么key需要可以通过参数来配置。这里刚好可以用上之前看的面向对象的知识。key作为实例的属性，方法挂载在原型上，于是就有了：

```javascript
function Store(STORAGE_KEY){
  // 私有属性
  this.STORAGE_KEY = STORAGE.KEY;
}

Store.prototype = {
  // 公共方法
  set: function(val){
    localStorage.setItem(this.STORAGE_KEY, JSON.stringify(val));
  },
  get: function(){
    JSON.parse(localStorage.getItem(this.STORAGE_KEY));
  },
  remove: function(){
    localStorage.removeItem(this.STORAGE_KEY);
  },
  clear: function(){
    localStorage.clear();
  }
}
```

这样，`store.js`的雏形就有了，使用的时候，先创建一个实例，给这个实例传入一个`STORAGE_KEY`，之后在这个实例上执行`set`操作就是把数据绑定到这个`STORAGE_KEY`下了。

```javascript
var store1 = new Store('store1');
store1.set({'name': 'Niko'});

var store2 = new Store('store2');
store2.set('name': 'Antony');

console.log(localStorage);  // store1: "{'name': 'Niko'}", store2: "{'name': 'Antony'}"
```

但是上面`Store`的定义方式是有一定的问题的。每个函数在创建的时候，都会默认的为它创建一个`prototype`对象，这个`prototype`对象的构造器属性`constructor`又指向构造函数，也就是这里的`Store`。现在`Store.prototype = {}`的这种写法完全重写了默认的`prototype`，新的`prototype`对象由于使用对象字面量的形式创建的，因此新的`prototype`对象是`Object`的实例，新的`prototype`对象的`constructor`属性也就指向了`Object`，梳理一下上面说的关系就是：

```javascript
// Store构造函数创建的时候：
Store.prototype = 默认的prototype对象
Store.prototype.constructor = Store

// 执行Store.prototype = {...}这个操作后：
Store.prototype = 新的prototype对象
Store.prototype.constructor = Object
```

以上这种情况不是我们想要的，我们可以手动的把`constructor`指回来,把最开始的代码简单改动一下，加上一句代码即可：

```javascript
Store.prototype = {
  constructor: Store, // 加上这一句就行，其它保持不变
};
```

但是新的问题又来了：`constructor`属性本来是不可枚举的，这么操作之后，它就变成可枚举属性了。如果想要保持它的不可枚举，则删掉上面添加的那一句代码，改用`Object.defineProperty`方法来定义`constructor`属性：

```javascript
// 其他不变，添加下面的代码：
Object.defineProperty(Store.prototype, 'constructor', {
  value: Store,
  Enumerable: false
});
```

我在工作中曾经遇到过 在某一项现有的数据下删除指定的某一项 的需求，于是把这个方法也加上了。基本思路就是：先`get`把数据取出来，然后`delete`掉指定的那一项后,再`set`存回去，应该很简单，如下：

```javascript
Store.prototype = {
  // other methods...
  removeItem: function(key){
    var getStore = this.get();
    delete getStore.key;  
    this.set(getStore);
  }
}
```

另外，如果在实例化的时候没有传入参数，那么`STORAGE_KEY`则为`undefined`，这显然不够友好，但是这里又不适合给参数设置一个默认的值。所以，如果没有传入参数，我让它抛出一个错误：

```javascritp
function Store(STORAGE_KEY) {
  if(STORAGE_KEY === '' || STORAGE_KEY === undefined){
    throw new Error('STORAGE_KEY can not be undefined');
  }
  this.STORAGE_KEY = STORAGE_KEY;
}
```

虽然功能都比较简单，但我还是把它放在了[我的github](https://github.com/nikolausliu/my-encapsulation/tree/master/store.js)上:)


<!-- more -->