---
title: 【手撕源码系列】手写Promise
date: 2021-11-05 18:01:18
tags:
    - JavaScript
categories:
    - 手撕源码
keywords: "面试,js基础"
description: 
cover: https://img.cdn.sugarat.top/mdImg/MTY0NjcwNDYzODcwMA==646704638700



---

# 手写Promise

这篇文章不会讲解Promise，所以实现的前提是对Promise有一个初步的了解

我们知道一个Promise执行的过程中有三个状态

没执行之前是`pending`、执行了`resolve`状态会变为`fulfilled`、执行了`reject`状态会变为`rejected`

我们也知道Promise只以第一次为准，第一次成功就永久为`fulfilled`，失败就为`rejected`

并且如果Promise有throw，就相当于执行了`reject`

## 实现resolve和reject

```js
class MyPromise {
    // 构造方法
    constructor(executor) {
        // 初始化值
        this.initBind()
        this.iniValue()
        // 执行传进来的参数
        executor(this.resolve,this.reject)
    }

    initBind(){
        // 初始化this
        this.resolve = this.resolve.bind(this)
        this.reject = this.reject.bind(this)
    }

    iniValue(){
        // 初始化值
        this.PromiseResult = null
        this.PromiseState = "pending"
    }

    resolve(reason){
        // 如果执行resolve 状态变为fulfilled
        this.PromiseState = "fulfilled"
        this.PromiseResult = reason
    }

    reject(reason){
        // 如果执行reject 状态变为rejected
        this.PromiseState = "rejected"
        this.PromiseResult = reason
    }
}


// 测试一下
let test = new MyPromise((resolve,reject)=>{
    resolve("成功")
})

console.log(test)

```

运行结果如图

![图片](https://img.cdn.sugarat.top/mdImg/MTY0NzM0NTE2MDkyNg==647345160926)

至此我们已经实现了一个基础版本，但是细心的同学会发现我们这个还是有点儿问题的。我们知道Promise的状态一旦改变就不能再改变了，但是这里如果我们连续调用的话是会改变状态的

```js
let test = new MyPromise((resolve,reject)=>{
    resolve("成功")
    reject("失败")
})

console.log(test)
```

![图片](https://img.cdn.sugarat.top/mdImg/MTY0NzM0NjE3NjU4MQ==647346176581)

可以看到我们的状态在`fulfilled`之后又改变为`rejected`这个是不被允许的

在之前的代码上做一个小小的改动

```js
resolve(reason){
        if(this.PromiseState !== "pending")  return 
        // 如果执行resolve 状态变为fulfilled
        this.PromiseState = "fulfilled"
        this.PromiseResult = reason
    }

    reject(reason){
        if(this.PromiseState !== "pending")  return
        // 如果执行reject 状态变为rejected
        this.PromiseState = "rejected"
        this.PromiseResult = reason
    }
```

一个小小的if判断，就可以解决掉了。那么我们还说过throw相当于调用reject，那么我们在执行executor的时候加一个try-catch

主体的Promise就实现的差不多了

## 实现then

- then接收两个回调，一个是`成功回调`，一个是`失败回调`
- 当Promise状态为`fulfilled`执行`成功回调`，为`rejected`执行`失败回调`
- 若`resolve`或`rejected`在定时器里面，则定时器结束后再执行`then`
- then支持`链式调用`，下一次执行受上一次返回值的影响

```js
then(onFulfilled,onRejected){
        // 接收两个回调
        // 确保收到的两个回调是函数
        onFulfilled = typeof onFulfilled === "function" ? onFulfilled : val => val
        onRejected = typeof onRejected === "function" ? onRejected : reason => {throw reason}

        if(this.PromiseState === "fulfilled"){
            // 如果当前为成功状态，执行第一个回调
            onFulfilled(this.PromiseResult)
        }else if(this.PromiseState === "rejected"){
            // 如果当前为失败状态，执行第二个回调
            onRejected(this.PromiseResult)
        }
    }
```

## 处理定时器的效果

如果我们在Promise中使用定时器的话，那么then的结果也是在定时器之后执行的，我们可以把then中的回调函数先保存起来，等过定时器时间再去判断执行哪一个回调。那么怎么保存这些回调呢？建议使用`数组`，因为一个Promise实例可能会多次then，用一个数组就一个一个保存了

```js
	iniValue(){
        // 初始化值
        this.PromiseResult = null
        this.PromiseState = "pending"
        this.onFulfilledCallbacks = []
        this.onRejectedCallbacks = []
    }

    resolve(reason){
        if(this.PromiseState !== "pending")  return
        // 如果执行resolve 状态变为fulfilled
        this.PromiseState = "fulfilled"
        this.PromiseResult = reason
        while(this.onFulfilledCallbacks.length){
            this.onFulfilledCallbacks.shift()(this.PromiseResult)
        }
    }

    reject(reason){
        if(this.PromiseState !== "pending")  return
        // 如果执行reject 状态变为rejected
        this.PromiseState = "rejected"
        this.PromiseResult = reason
        while(this.onRejectedCallbacks.length){
            this.onRejectedCallbacks.shift()(this.PromiseResult)
        }
    }

    then(onFulfilled,onRejected){
        // 接收两个回调
        // 确保收到的两个回调是函数
        onFulfilled = typeof onFulfilled === "function" ? onFulfilled : val => val
        onRejected = typeof onRejected === "function" ? onRejected : reason => {throw reason}

        if(this.PromiseState === "fulfilled"){
            // 如果当前为成功状态，执行第一个回调
            onFulfilled(this.PromiseResult)
        }else if(this.PromiseState === "rejected"){
            // 如果当前为失败状态，执行第二哥回调
            onRejected(this.PromiseResult)
        }else if(this.PromiseState === "pending"){
            this.onFulfilledCallbacks.push(onFulfilled.bind(this))
            this.onRejectedCallbacks.push(onRejected.bind(this))
        }
    }
```

当然我们还得去实现链式调用

- then方法本身会返回一个新的Promise对象
- 如果返回值是promise对象，返回值为成功，新promise就是成功
- 如果返回值是promise对象，返回值为失败，新promise就是失败
- 如果返回值非promise对象，新promise对象就是成功，值为此返回值

```js
class MyPromise {
    // 构造方法
    constructor(executor) {
        // 初始化值
        this.initBind()
        this.iniValue()
        // 执行传进来的参数
        try {
            executor(this.resolve,this.reject)
        }catch (e) {
            this.reject(e)
        }
    }

    initBind(){
        // 初始化this
        this.resolve = this.resolve.bind(this)
        this.reject = this.reject.bind(this)
    }

    iniValue(){
        // 初始化值
        this.PromiseResult = null
        this.PromiseState = "pending"
        this.onFulfilledCallbacks = []
        this.onRejectedCallbacks = []
    }

    resolve(reason){
        if(this.PromiseState !== "pending")  return
        // 如果执行resolve 状态变为fulfilled
        this.PromiseState = "fulfilled"
        this.PromiseResult = reason
        while(this.onFulfilledCallbacks.length){
            this.onFulfilledCallbacks.shift()(this.PromiseResult)
        }
    }

    reject(reason){
        if(this.PromiseState !== "pending")  return
        // 如果执行reject 状态变为rejected
        this.PromiseState = "rejected"
        this.PromiseResult = reason
        while(this.onRejectedCallbacks.length){
            this.onRejectedCallbacks.shift()(this.PromiseResult)
        }
    }

    then(onFulfilled,onRejected){
        // 接收两个回调
        // 确保收到的两个回调是函数
        onFulfilled = typeof onFulfilled === "function" ? onFulfilled : val => val
        onRejected = typeof onRejected === "function" ? onRejected : reason => {throw reason}

        let thenPromise = new MyPromise((resolve,reject)=>{

            const resolvePromise = cb =>{
                try {
                    const x = cb(this.PromiseResult)
                    if(x === thenPromise){
                        // 不能等于自身
                        throw new Error("不能返回自身")
                    }
                    if(x instanceof MyPromise){
                        // 如果返回值是Promise
                        // 如果返回值是promise对象，返回值为成功，新promise就是成功
                        // 如果返回值是promise对象，返回值为失败，新promise就是失败
                        // 谁知道返回的promise是失败成功？只有then知道
                        x.then(resolve,reject)
                    }else{
                        // 非Promise就是成功
                        resolve(x)
                    }
                }catch (e) {
                    reject(e)
                }
            }

            if(this.PromiseState === "fulfilled"){
                // 如果当前为成功状态，执行第一个回调
                resolvePromise(onFulfilled)
            }else if(this.PromiseState === "rejected"){
                // 如果当前为失败状态，执行第二哥回调
                resolvePromise(onRejected)
            }else if(this.PromiseState === "pending"){
                this.onFulfilledCallbacks.push(resolvePromise.bind(this,onFulfilled))
                this.onRejectedCallbacks.push(resolvePromise.bind(this,onRejected))
            }
        })

        return thenPromise
    }
}
```

目前基本的Promise实现的差不多了后续还有其他的几个方法后续再实现...【未完待续...】

