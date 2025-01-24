---
title: 【React深入学习(一)】JSX到底是什么？
date: 2022-2-10 12:58:18
tags:
    - React
categories:
    - React源码
keywords: "框架,React"
description: React的深入浅出
cover: https://img.cdn.sugarat.top/mdImg/MTY0NzkxNDI5ODYxOA==647914298618
---

## JSX的本质：JavaScript的扩展

我们可以看看[react官网](https://react.docschina.org/docs/glossary.html#jsx)给的定义

> JSX 是一个 JavaScript 语法扩展。它类似于模板语言，但它具有 JavaScript 的全部能力

**语法扩展**很好理解，**充具备JavaScript的能力**这句话该如何理解？也就是说**JSX语法如何在JavaScript中生效**？

## 初识babel

Facebook 公司给 JSX 的定位是 JavaScript 的“扩展”，而非 JavaScript 的“某个版本”，这就直接决定了浏览器并不会像天然支持 JavaScript 一样地支持 JSX。那么，JSX 的语法是如何在 JavaScript 中生效的呢？[React 官网](https://react.docschina.org/docs/glossary.html#jsx)其实早已给过我们线索：

> JSX 会被编译为 React.createElement()， React.createElement() 将返回一个叫作“React Element”的 JS 对象。

这个“编译”是怎么回事：“编译”这个动作，是由 Babel 来完成的。

## 什么是Babel？

> 是一个工具链，主要用于将 ECMAScript 2015+ 版本的代码转换为向后兼容的 JavaScript 语法，以便能够运行在当前和旧版本的浏览器或其他环境中。
>
> ——Babel 官网

比如说，ES2015+ 版本推出了一种名为“模板字符串”的新语法，这种语法在一些低版本的浏览器里并不兼容。下面是一段模板字符串的示例代码：

```js
var name = "Guy Fieri";

var place = "Flavortown";

`Hello ${name}, ready for ${place}?`;

```

Babel 就可以帮我们把这段代码转换为大部分低版本浏览器也能够识别的 ES5 代码：

```js
var name = "Guy Fieri";

var place = "Flavortown";

"Hello ".concat(name, ", ready for ").concat(place, "?");

```

类似的，**Babel也具有将JSX语法转换为JavaScript代码的能力**

我们知道了babel处理高级版本语法的JavaScript，那么babel处理JSX是什么样的呢？

我们可以在[Babel官网](https://www.babeljs.cn/repl#?browsers=defaults%2C%20not%20ie%2011%2C%20not%20ie_mob%2011&build=&builtIns=false&corejs=3.6&spec=false&loose=false&code_lz=DwEwlgbgBAxgNgQwM5IHIILYFMoF4oBEAggA4kEB8AUFFMABYCMsiK62ehALmF3FpQCSUTFC70cPPlmAB6JtVrASLZGkw58BGAHsAdlywGhIjGImx9hg3JLU54CBSA&debug=false&forceAllTransforms=false&shippedProposals=false&circleciRepo=&evaluate=false&fileSize=false&timeTravel=false&sourceType=module&lineWrap=true&presets=env%2Creact%2Cstage-2&prettier=false&targets=&version=7.17.8&externalPlugins=&assumptions=%7B%7D)测试一下如下代码会被转化为什么结果

```jsx
<div className = "App">
  <h1 className = "title">I am the title</h1>
  <p className = "content">I am the content</p>
</div>
```

结果

![图片](https://img.cdn.sugarat.top/mdImg/MTY0NzgyOTU1NzM2OQ==647829557369)

可以看到，所有的 JSX 标签都被转化成了 React.createElement 调用，这也就意味着，我们写的 JSX 其实写的就是 React.createElement，虽然它看起来有点像 HTML，但也只是“看起来像”而已。J**SX 的本质是React.createElement这个 JavaScript 调用的语法糖**，这也就完美地呼应上了 React 官方给出的“**JSX 充分具备 JavaScript 的能力**”这句话。

## 为什么React会用JSX语法？

那为什么不直接使用React.createElement来创建元素呢？

其实在前后对比图一出来的时候你就应该会发现，在实际功能一直的前提下，JSX 代码层次分明、嵌套关系清晰；而 React.createElement 代码则给人一种非常混乱的“杂糅感”，这样的代码不仅读起来不友好，写起来也费劲。

JSX 语法糖允许前端开发者使用我们最为熟悉的类 HTML 标签语法来创建虚拟 DOM，在降低学习成本的同时，也提升了研发效率与研发体验

## JSX是如何映射为DOM的

我们先来看看React.createElement的部分源码

```js
/**
 101. React的创建元素方法
 */
export function createElement(type, config, children) {
  // propName 变量用于储存后面需要用到的元素属性
  let propName; 
  // props 变量用于储存元素属性的键值对集合
  const props = {}; 
  // key、ref、self、source 均为 React 元素的属性，此处不必深究

  let key = null;
  let ref = null; 
  let self = null; 
  let source = null; 

  // config 对象中存储的是元素的属性
  if (config != null) { 
    // 进来之后做的第一件事，是依次对 ref、key、self 和 source 属性赋值
    if (hasValidRef(config)) {
      ref = config.ref;
    }
    // 此处将 key 值字符串化
    if (hasValidKey(config)) {
      key = '' + config.key; 
    }
    self = config.__self === undefined ? null : config.__self;
    source = config.__source === undefined ? null : config.__source;
    // 接着就是要把 config 里面的属性都一个一个挪到 props 这个之前声明好的对象里面
    for (propName in config) {
      if (
        // 筛选出可以提进 props 对象里的属性
        hasOwnProperty.call(config, propName) &&
        !RESERVED_PROPS.hasOwnProperty(propName) 
      ) {
        props[propName] = config[propName]; 
      }
    }
  }
  // childrenLength 指的是当前元素的子元素的个数，减去的 2 是 type 和 config 两个参数占用的长度
  const childrenLength = arguments.length - 2; 
  // 如果抛去type和config，就只剩下一个参数，一般意味着文本节点出现了
  if (childrenLength === 1) { 
    // 直接把这个参数的值赋给props.children
    props.children = children; 
    // 处理嵌套多个子元素的情况
  } else if (childrenLength > 1) { 
    // 声明一个子元素数组
    const childArray = Array(childrenLength); 
    // 把子元素推进数组里
    for (let i = 0; i < childrenLength; i++) { 
      childArray[i] = arguments[i + 2];
    }
    // 最后把这个数组赋值给props.children
    props.children = childArray; 
  } 

  // 处理 defaultProps
  if (type && type.defaultProps) {
    const defaultProps = type.defaultProps;
    for (propName in defaultProps) { 
      if (props[propName] === undefined) {
        props[propName] = defaultProps[propName];
      }
    }
  }


  // 最后返回一个调用ReactElement执行方法，并传入刚才处理过的参数
  return ReactElement(
    type,
    key,
    ref,
    self,
    source,
    ReactCurrentOwner.current,
    props,
  );
}
```

当然这个看起来确实有点儿不好理解，总的来说就是将传入的参数`type`,`config`,`children`解析成一个DOM

createElement 有 3 个入参，这 3 个入参囊括了 React 创建一个元素所需要知道的全部信息。

- type：用于标识节点的类型。它可以是类似“h1”“div”这样的标准 HTML 标签字符串，也可以是 React 组件类型或 React fragment 类型。
- config：以对象形式传入，组件所有的属性都会以键值对的形式存储在 config 对象中。
- children：以对象形式传入，它记录的是组件标签之间嵌套的内容，也就是所谓的“子节点”“子元素”。

```React
React.createElement("ul", {
  // 传入属性键值对
  className: "list"
   // 从第三个入参开始往后，传入的参数都是 children
}, React.createElement("li", {
  key: "1"
}, "1"), React.createElement("li", {
  key: "2"
}, "2"));
```

其实就是下面的DOM结构

```html
<ul className="list">
  <li key="1">1</li>
  <li key="2">2</li>
</ul>
```

接下来我们看一下React.createElement整体流程

![图片](https://img.cdn.sugarat.top/mdImg/MTY0Nzg0OTA0NDA3OQ==647849044079)

这么一看，结合上面的源码，会发现React.createElement做的工作也是比较清晰的，它更像是一种数据转换器，作为开发人员与reactElement之间

![图片](https://img.cdn.sugarat.top/mdImg/MTY0Nzg0OTUyODQxOQ==647849528419)

上面已经分析过，createElement 执行到最后会 return 一个针对 ReactElement 的调用。下面是ReactElement的部分源码

```js
const ReactElement = function(type, key, ref, self, source, owner, props) {
  const element = {
    // REACT_ELEMENT_TYPE是一个常量，用来标识该对象是一个ReactElement
    $$typeof: REACT_ELEMENT_TYPE,
    // 内置属性赋值
    type: type,
    key: key,
    ref: ref,
    props: props,
    // 记录创造该元素的组件
    _owner: owner,
  };
  // 
  if (__DEV__) {
    // 这里是一些针对 __DEV__ 环境下的处理，对于大家理解主要逻辑意义不大，此处我直接省略掉，以免混淆视听
  }
  return element;
};
```

从逻辑上我们可以看出，ReactElement 其实只做了一件事情，那就是“**创建**”，说得更精确一点，是“**组装**”：ReactElement 把传入的参数按照一定的规范，“组装”进了 element 对象里，并把它返回给了 React.createElement，最终 React.createElement 又把它交回到了开发者手中

![图片](https://img.cdn.sugarat.top/mdImg/MTY0Nzg0OTcxMjU2MQ==647849712561)

尝试输出我们下面示例App'组件的 JSX 部分来验证一下我们的结论

```react
const AppJSX = (<div className="App">
  <h1 className="title">I am the title</h1>
  <p className="content">I am the content</p>
</div>)
console.log(AppJSX)
```

结果如下

![图片](https://img.cdn.sugarat.top/mdImg/MTY0Nzg0OTg5MjA3MA==647849892070)

这个 ReactElement 对象实例，本质上是**以 JavaScript 对象形式存在的对 DOM 的描述**，也就是老生常谈的**“虚拟 DOM”**，既然是“虚拟 DOM”，那就意味着和渲染到页面上的真实 DOM 之间还有一些距离，这个“距离”，就是由大家喜闻乐见的**ReactDOM.render**方法来填补的

在每一个 React 项目的入口文件中，都少不了对 React.render 函数的调用。下面我简单介绍下 ReactDOM.render 方法的入参规则

```js
ReactDOM.render(
// 需要渲染的元素（ReactElement）
element, 
// 元素挂载的目标容器（一个真实DOM）
container,
// 回调函数，可选参数，可以用来处理渲染结束后的逻辑
[callback])
```

ReactDOM.render 方法可以接收 3 个参数，其中**第二个参数就是一个真实的 DOM 节点**，**这个真实的 DOM 节点充当“容器”的角色**，React 元素最终会被渲染到这个“容器”里面去。

