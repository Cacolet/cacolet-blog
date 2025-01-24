---
title: 【手撕源码系列】实现new、继承
date: 2021-11-08 19:01:18
tags:
    - JavaScript
categories:
    - 手撕源码
keywords: "面试,js基础"
description: 
cover: https://img.cdn.sugarat.top/mdImg/MTY0NjcwNDYzODcwMA==646704638700

---

# 实现new、继承

## new的实现

我们先看一看new是在干嘛，下面是一段正常的new

```js
function Person(name, age) {
    this.name = name
    this.age = age
}
let p = new Person('cacolet', 20)
console.log(p)  // Person { name: 'cacolet', age: 20 }

```

我们打印实例p可以看到

![图片](https://img.cdn.sugarat.top/mdImg/MTY0NzQ4MDA3NDI5NA==647480074294)

用new关键字实例化对象时，首先创建了一个空对象p，并且这个空对象包含两个属性name和age，分别对应构造函数中的两个属性，其次我们也可以知道实例化出来的这个对象p是继承自Person.prototype,那么现在我们就可以总结出new关键字在实例化对象时内部都干了什么

- 创建一个空对象，并使该空对象继承constructor.prototype
- 执行构造函数，并将this指向刚刚创建的新对象
- 返回新对象

```js
function Animal(type) {
    this.type = type
}

Animal.prototype.say = function () {
    console.log('汪汪')
}

function mockNew() {
    let Constructor = [].shift.call(arguments) // 取出构造函数Animal
    let obj = {} // 创建一个新对象
    obj.__proto__ = Constructor.prototype  // 继承构造函数的原型对象
    Constructor.apply(obj,arguments)  // 改变this的指向
    return obj
}

let animal = mockNew(Animal,'dog')
console.log(animal.type) // dog
animal.say() // 汪汪
```

一个new就实现了

## 实现继承

继承实现篇主要是根据《JavaScript高级程序设计》做的总结

并且实现的过程跟原型链有很大的关系，这块知识不熟悉的同学，建议先去补补原型链的知识

### 1.原型链继承

```js
/*
* 原型链继承
* */

function Parent(){
    this.name = ['cacolet'];
    this.age = 20
}

Parent.prototype.getName = function () {
    console.log(this.name)
}

function Child() {}

Child.prototype = new Parent()
let child = new Child()
child.getName() // ['cacolet']
```

缺点：

1. 引用类型的属性所有实例共享，不明白的看例子

   ```js
   let child1 = new Child()
   child1.name.push('2022')
   let child1 = new Child()
   child.getName() // ['cacolet','2022']
   ```

   可以发现实例之间是共享引用的属性的，这不符合继承的特性

2. 在创建Child的实例时，不能向Parent传参

### 2.利用构造函数

```js
/*
* 借用构造函数()
* */
function Parent(name) {
     this.name = name
}

function Child(name) {
    Parent.call(this,name) // 每次实例化的时候都要执行
}

let child = new Child('cacolet')
console.log(child.name)

```

这样虽然实现了可以进行参数的传递，父类的引用属性不会被共享

缺点：

子类不能访问父类的原型方法，因此所有方法属性都写在构造函数中，每次实力化都会初始化

### 3.组合继承

组合继承综合了`原型链继承`和`构造函数继承`，将两者的优点结合起来

基本思路是使用原型链继承原型上的属性和方法，而通过构造函数继承实例属性，这样既可以把方法定义在原型上以实现重用，又可以让每个实例都有自己的属性

```js
/*
* 组合继承
* */
function Parent(name) {
    this.name = name
    this.colors = ["red","blue"]
}

Parent.prototype.sayName = function () {
    console.log(this.name)
}

function Child(name,age) {
    // 继承父类属性
    Parent.call(this,name)
    this.age = age
}

// 继承父类方法
Child.prototype = new Parent()

Child.prototype.sayAge = function () {
    console.log(this.name)
}


let child1 = new Child("cacolet",20)

console.log(child1)
```

这样实例不仅拥有自己的属性的，也能共享父类原型上的属性，也是比较常用的javascript的继承方法

### 4.原型式继承

就是 ES5 Object.create 的模拟实现，将传入的对象作为创建的对象的原型。

```js
/*
* 原型式继承
* */

function createObj(obj){
    function F() {}
    F.prototype = obj;
    return new F()
}

let person = {
    name : 'cacolet',
    color: ['red','blue']
}

let person1 = createObj(person)
let person2 = createObj(person)
person1.name = 'xxx'
console.log(person1.name) // 'xxx'
// 注意这里person1.name 是给person1新增加了一个属性
// 并不是修改了person上的name
console.log(person1.__proto__.name) // 'cacolet'
person1.color.push('yellow')
console.log(person2.color) // [ 'red', 'blue', 'yellow' ]
```

### 5.寄生式继承

其实就是用原型式继承对一个目标对象进行浅复制，增强这个浅复制的能力，这里直接中Object.create来实现

```js
/*
* 寄生式继承
* */

function createAnother(obj) {
    let clone = Object.create(obj)
    clone.getName = function () {
        console.log(this.name)
    }
}
```

缺点：

- 跟借用构造函数模式一样，每次创建对象都会创建一遍方法
- 原型链继承多个实例的引用类型属性指向相同，存在篡改的可能。
- 无法传递参数

### 6.寄生组合式继承

```js
/*
* 寄生组合式继承
* */
function Parent (name) {
    this.name = name;
    this.colors = ['red', 'blue', 'green'];
}

Parent.prototype.getName = function () {
    console.log(this.name)
}

function Child (name, age) {
    Parent.call(this, name);
    this.age = age;
}

// 关键的三步
var F = function () {};

F.prototype = Parent.prototype;

Child.prototype = new F();


var child1 = new Child('kevin', '18');

console.log(child1);
```

然后做一个简单的封装

```js
function object(o) {
    function F() {}
    F.prototype = o;
    return new F();
}

function prototype(child, parent) {
    var prototype = object(parent.prototype);
    prototype.constructor = child;
    child.prototype = prototype;
}

// 当我们使用的时候：
prototype(Child, Parent);

```

完结...

