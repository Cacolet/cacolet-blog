---
title: 【手撕源码系列】call、apply、bind
date: 2021-10-30 10:01:18
tags:
    - JavaScript
categories:
    - 手撕源码
keywords: "面试,js基础"
description: 
cover: https://img.cdn.sugarat.top/mdImg/MTY0NjcwNDYzODcwMA==646704638700
---

# call、apply、bind实现

在实现原理之前，我们要知道这三个方法的作用是什么，首先得了解this的指向规则(this的指向)，改变this的指向是这三个方法的主要用处。

## 一、call方法

参数第一个为指定的对象，后面为参数列表

call 格式 [function].call([this], [param]...)，一句话概括：`call()` 将函数的 `this` 指定到 `call()` 的第一个参数值同时将剩余参数指定的情况下调用某个函数或方法。

实现步骤：

- 将函数设为对象的属性
- 执行、删除这个函数
- 指定this到函数并传入给定的执行参数
- 如果不传入参数，默认指向window

```js
Function.prototype.myCall = function(context,...args){
    if(typeof this !== "function") {
            throw new TypeError('Error')
        }
    // 这里是实现第一参数没有的情况下调用指向window
    if(!context || context === null){ context = window }
    // 这里context如果已经有fn属性了 那么这时候是会影响的
    // 可以使用ES6提供的Symbol
    // let fn = Symbol();
    // context[fn] = this; 
    context.fn = this // 这里this指向的是调用者 谁调用就指向那个函数
    // 关于实现参数的传递 有ES6提供的扩展运算符 
    // 因为我们知道call后面跟的是参数列表
    // 所以args在...的作用下就是数组 所以在后面context.fn中调用在将数组展开...args
    let result = context.fn(...args)
    delete context.fn
    return result
}
```

`call`的实现相对简单，注意参数的处理和this指向就对了，当然我们也可以不使用扩展运算符`...`，我们可以使用`eval`这个很变态的方法

```js
Function.prototype.myCall = function(context){
    if(typeof this !== "function") {
            throw new TypeError('Error')
        }
    // 这里在定义的时候直接定义成context，后面的参数我们可以通过arguments对象获取
    if(!context){ context = window }
    context.fn = this
    var args = []
    // 这里通过将arguments拆解为我们需要的参数，并以字符串的形式存储在数组中
    // eval会将字符串执行，执行结果返回
    for(var i = 1, len = arguments.length;i< len;i++){
    	args.push('arguments['+i+']')
    }
    // 小提示：注意'123' + ['a','b'] + '321'的结果哟 =>'123a,b321'
    let result = eval('context.fn(' + args + ')')
    delete context.fn
    return result
}
```

## 二、apply方法

参数第一个为指定的对象，后面为数组

`apply`格式 [function].call([this], [param])，一句话概括：`apply()` 将函数的 `this` 指定到 `apply()` 的第一个参数值同时将第二个参数数组指定的情况下调用某个函数或方法。

实现步骤：

- 将函数设为对象的属性
- 执行、删除这个函数
- 指定this到函数并传入给定的执行参数
- 如果不传入参数，默认指向window

```js
Function.prototype.myApply = function(context,args){
    if(typeof this !== "function") {
            throw new TypeError('Error')
        }
    if(!context) { context = window }
    // context = context || window
    context.fn = this
    // 提取参数
    let result = context.fn(...args)
    delete context.fn
    return result
}
```

当然也可以用eval，与上面call一样

```js
Function.prototype._Apply = function (context) {
        if(typeof this !== "function") {
            throw new TypeError('Error')
        }
        context = context || window
        context.fn = this
        if(!arguments[1]){
            result = context.fn()
        }else{
            let args = []
            for(var i = 0;i< arguments[1].length;i++){
                 args.push('arguments[1]['+i+']')
            }
            let result = eval('context.fn('+args+')')
        }
        delete context.fn
        return result
    }
```

## 三、bind方法

`bind()` 方法会创建一个新函数。当这个新函数被调用时，`bind()` 的第一个参数将作为它运行时的 `this`，之后的一序列参数将会在传递的实参前传入作为它的参数。

实现步骤

- 返回一个新函数，新函数执行的this才指定bind的第一个参数
- bind的剩余参数，传递给新的函数
- 返回后的函数是自我调用，且可以接受新的参数
- 通过new调用时，this指向为实例化对象

```js
Function.prototype.myBind = function(context){
    if(typeof this !== "function") {
            throw new TypeError('Error')
        }
    // 这里的this指向的是最初的函数，fn.bind()中的fn函数
    let self = this;
    let args = Array.prototype.slice.call(arguments,1)
    let FN = function(){}
    let FB = function(){
        // 新函数的参数的提取
        let bindArgs = Array.prototype.slice.call(arguments)
        // 这里是重点和难点
        // 这里的this指向的是返回的新函数的调用环境，这里FN和FB都是可以的
        // 因为下面的FB的原型是通过FN这个构造函数的实例化来的
        // 并且这里的this要么指向new过后的实例化对象，要么就是调用这个新函数的对象
        // 如果是true那就是实例化对象
        // 然后将最初的函数在this指向的环境下(实例化或者context)调用
        return self.apply(this instanceof FB ? this :context,args.concat(bindArgs))
        // 这里有点儿绕，总的来说就是判断self调用的环境，要么是new后的对象，要么是最初的想		 // 绑定的context
    }
    // 这里是为了避免后续在修改FB.prototype的时候修改了最初的函数也就是self的prototype
   	FN.prototype = this.prototype
    FB.prototype = new FN()
    return FB
}
```

到这里三个方法就实现的差不多了，至少目前是够用了。
