---
title: 【JavaScript】数组API
date: 2021-10-04 16:58:18
tags:
    - JavaScript
    - 数组
categories:
    - JavaScript
keywords: "面试,js基础"
description: 总结ES5、ES6的数组方法
cover: https://img.cdn.sugarat.top/mdImg/MTY0Njc5Mzk4MjMwMg==646793982302
---

# 数组API

## ES5中的数组方法

### indexOf

> 用于查找数组中是否存在某个值，存在返回下标，不存在返回`-1`

```js
let list = [1,2,3]
list.indexOf(1) // 0
list.indexOf(cacolet) // -1
```

### map

>用来遍历数组并返回新数组，不改变原数组
>
>接受三个参数(value,index,self)
>
>value:数组值 index:数组下标 self:自身数组
>
>只有一个参数时，默认为value

```js
let list = [1,2,3]
let res = list.map((value,index,self)=>{
    // 改变value是不会改变list的值
    value = 1
    
    // 改变self是会改变list的
    self[0] = 9 // 此时list的值 [9,2,3]
    
    // 如果没有return res的值为undefined
    return value + 1
})

res // [2,2,2]
list // [9,2,3]
```

### forEach

> 用来遍历数组,没有返回值
>
> 接受三个参数(value,index,self)
>
> value:数组值 index:数组下标 self:自身数组
>
> 只有一个参数时，默认为value

```js
let list = [1,2,3]
let res = list.forEach((value,index,self) =>{
    // value = 1 不改变原数组
    // self[0] = 9 改变原数组
    return 123 // 返回值无法被接受 res的值为undefined
})
```

### splice

> 用于删除或者替换内容，会改变原数组，返回值为删除或替换掉的元素
>
> 接受三个参数(start,num,...args)
>
> start:起始位置  num:修改的位数  args:需要添加的元素

```js
let list = [1,2,3,4,5]
let res = list.splice(1,2,0,0)
res // [2,3]
list // [1,0,0,4,5]
```

### slice

> 用于截取数组，返回值为截取的元素数组
>
> 接受两个参数(start,end)
>
> start:起始位置(下标) end:结束位置(下标)
>
> end没有的值时，截取到最后

```js
let list = [1,2,3,4,5]
let res0 = list.slice(1,3) // [2,3] 不包含3下标对应的值
let res1 = list.slice(1) // [2,3,4,5]
let res2 = list.slice() // 默认从0开始到末尾  相当于深拷贝一个数组
```

### filter

> 用于过滤数组内符合条件的值，返回满足条件的数组对象，不改变原数组
>
> 接受两个参数(callback,this)
>
> callback回调函数
>
>  this:可选。对象作为该执行回调时使用，传递给函数，用作 "this" 的值。
> 如果省略了 thisValue ，"this" 的值为 "undefined"
>
> callback的三个参数(value,index,self)
>
> value:当前元素的值
>
> index:下标
>
> self:自身函数

```js
let list = [1,2,3]
let res = list.filter((item) =>{
    return item > 1
})
res // [2,3]
list // [1,2,3]
```

#### 贴：数组去重

```js
let arrFilter = list.filter((value,index,self)=>{
    return self.indexOf(value) === index
})
```

### every、some

#### every

> 用于检测数组所有元素是否符合指定条件返回值为`Boolean`,该方法是数组中必须全部值元素满足的条件
>
> 是返回true，否返回false
>
> 参数接受一个回调函数

```js
let list = [1,2,3]
let res = list.every(item => item > 0) // true
let res = list.every(item => item > 1) // false
```

#### some

> 与every用法一样
>
> 返回结果的判断方式不同
>
> 只要有满足条件的就是true，全部不满足条件就是false

```js
let list = [1,2,3]
let res = list.every(item => item < 0) // false
let res = list.every(item => item > 1) // true
```

### reduce(比较重要)

>接受两个参数，第一个回调函数，第二个位回调函数中pre的初始值
>
>回调函数的四个参数(pre,cur,index,self)
>
>pre:计算结束返回的值
>
>cur:当前元素
>
>index:当前元素的索引
>
>self:数组本身

```js
let list = [1,2,3]

let res = list.reduce((prev,cur) => prev += cur,0) // 6
```

### reverse

> 反转数组
>
> 返回新数组，并且改变原数组

```js
let list = [1,2,3]
let res = list.reverse() // [3,2,1]
res // [3,2,1]
list // [3,2,1]
```

### join

> 拼接数组
>
> 返回一个字符串，不改变原数组

``` js
let list = [1,2,3]
let res1 = list.join('') // 123
let res2 = list.join(' ')// 1 2 3
let res3 = list.join('-')// 1-2-3
```

### sort

> 排序 返回新数组并且也会改变原数组
>
> 可以接收一个回调函数作为参数
>
> 当排序的元素小于10个是用的是插入排序，当大于十个用的是快排

```js
let list = [1,2,3,11,22,33]
let sort1 = list.sort() // 默认为字典序 [1,11,2,22,3,33]
let sort2 = list.sort((a,b) => a - b)  // [1,2,3,11,22,33]
let sort3 = list.sort((a,b) => b - a)  // [33,22,11,3,2,1]
// 这个例子用的是一个数组，如果同时运行 那么结果只有一个，如果想得到正确的结果，请设计多个数组
```

### concat

> 用于合并数组 返回值为数组
>
> 参数为数组和参数列表都是可以的

```js
let list = [1,2,3]
let res = list.concat([1,2,3]) // [1,2,3,1,2,3]
let res = list.concat([1,2,3],7,8,9) // [1,2,3,1,2,3,7,8,9]
let res = list.concat(0,7,8,9) // [1,2,3,0,7,8,9]
```

### push

> 数组后面添加元素，返回值为数组的length

```js
let list = [1,2,3]
let res = list.push(1,2,3)
res // 6
list // [1,2,3,1,2,3]
```

### pop

> 数组后面删除元素，返回值为删除的元素

```js
let list = [1,2,3]
let res = list.pop() // 3
res // 3
list // [1,2]
```

### shift

> 删除前面的元素，返回值为删除的元素

```js
let list = [1,2,3]
let res = list.shift() 
res // 1
list // [2,3]
```

### unshift

> 数组头部添加元素 返回值为数组的长度

```js
let list = [1,2,3]
let res = list.unshift(1)
res // 4
list // [1,1,2,3]
let res = list.unshift(7,8,9) // [7,8,9,1,2,3] 注意一个一个添加和多元素添加的顺序
```

### toString

> 将数组转化为字符串 数组不改变

```js
let list = [1,2,3]
let res = list.toString()
res // 1,2,3
```

### Array.isArray

> 检测对象是不是一个数组

```js
let list = [1,2,3]
let res = Array.isArray(list) // true
```

## ES6+

### includes

> 检测数组是否存在某值 返回Boolean

```js
let list = [1,2,3]
let res = list.includes(1) // true
let res = list.includes(0) // false
```

### find & findIndex

> 方法返回通过测试（函数内判断）的数组的第一个元素的值。不改变原始值
>
> ind() 方法为数组中的每个元素都调用一次函数执行：
>
> 当数组中的元素在测试条件时返回 *true* 时, find() 返回符合条件的元素，之后的值不会再调用执行函数。
>
> 如果没有符合条件的元素返回 undefined
>
> 语法：array.find(function(currentValue, index, arr),thisValue)

```js
let list = [1,2,3]
let res1 = list.find((item) =>{
    return item > 2
})
res1 // 3 返回值
let res2  list.findIndex((item) =>{
    return item > 2
})
res2 // 2 返回下标
```

### flat

> 扁平化数组，返回扁平化后的数组，不改变原数组
>
> 接受一个整数作为参数，表示扁平的层数

```js
let list = [1,2,3,[4,4,4,[5,5,5]]]
let res = list.flat() // [1,2,3,4,4,4,[5,5,5]]
let res = list.flat(2) // [1,2,3,4,4,4,5,5,5]
```

### fill

> 将一个固定值替换数组的元素，返回值为新数组，会改变原数组
>
> 三个参数(value,start,end)
>
> value：必需，填充的值
>
> start：起始位置
>
> end：结束位置，默认为最后一位

```js
let list = [1,2,3,4]
let res = list.fill('cacolet') // ['cacolet','cacolet','cacolet','cacolet']
```

### Array.from

> 将伪数组转化为真数组

```js
let res = Array.from(document.getElementsByTagName("div")) 
```

### 改变原始数组值的有哪些

`splice`、`reverse`、`sort`、`push`、`pop`、`shift`、`unshift`、`fill`、

------

以后还有的新内容会随时更新......【未完待续...】

