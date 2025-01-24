---
title: 【React深入学习(四)】React的数据流
date: 2022-2-16 12:58:18
tags:
    - React
categories:
    - React源码
keywords: "框架,React"
description: React的深入浅出
cover: https://img.cdn.sugarat.top/mdImg/MTY0NzkxNDI5ODYxOA==647914298618

---

## 基于 props 的单向数据流

所谓单向数据流，指的就是当前组件的 state 以 props 的形式流动时，**只能流向组件树中比自己层级更低的组件**。 比如在父-子组件这种嵌套关系中，只能由父组件传 props 给子组件，而不能反过来

### 父-子组件通信

原理讲解

这是最常见、也是最好解决的一个通信场景。React 的数据流是单向的，父组件可以直接将 this.props 传入子组件，实现父-子间的通信。

### 子-父组件通信

原理讲解

考虑到 props 是单向的，子组件并不能直接将自己的数据塞给父组件，但 props 的形式也可以是多样的。假如父组件传递给子组件的是一个绑定了自身上下文的函数，那么子组件在调用该函数时，就可以将想要交给父组件的数据以函数入参的形式给出去，以此来间接地实现数据从子组件到父组件的流动。

### 兄弟组件通信

原理讲解

兄弟组件之间共享了同一个父组件，如下图所示，这是一个非常重要的先决条件。

这个先决条件使得我们可以继续利用父子组件这一层关系，将“兄弟 1 → 兄弟 2”之间的通信，转化为“兄弟 1 → 父组件”（子-父通信）、“父组件 → 兄弟 2”（父-子）通信两个步骤，这样一来就能够巧妙地把“兄弟”之间的新问题化解为“父子”之间的旧问题。

## 利用“发布-订阅”模式驱动数据流

“发布-订阅”模式可谓是解决通信类问题的“万金油”，在前端世界的应用非常广泛，最为熟知的，应该还是 Vue.js 中作为常规操作被推而广之的“全局事件总线” EventBus。

### 理解事件的发布-订阅机制

发布-订阅机制早期最广泛的应用，应该是在浏览器的 DOM 事件中。  相信有过原生 JavaScript 开发经验的同学，对下面这样的用法都不会陌生：

```js
target.addEventListener(type, listener, useCapture);
```

通过调用 addEventListener 方法，我们可以创建一个事件监听器，这个动作就是“订阅”。比如我可以监听 click（点击）事件：

```
el.addEventListener("click", func, false);
```

这样一来，当 click 事件被触发时，事件会被“发布”出去，进而触发监听这个事件的 func 函数。这就是一个最简单的发布-订阅案例。

使用发布-订阅模式的优点在于，监听事件的位置和触发事件的位置是不受限的，就算相隔十万八千里，只要它们在同一个上下文里，就能够彼此感知。这个特性，太适合用来应对“任意组件通信”这种场景了。

### 发布-订阅模型 API 设计思路

通过前面的分析，不难看出发布-订阅模式中有两个关键的动作：事件的监听（订阅）和事件的触发（发布），这两个动作自然而然地对应着两个基本的 API 方法。

on()：负责注册事件的监听器，指定事件触发时的回调函数。

emit()：负责触发事件，可以通过传参使其在触发的时候携带数据 。

最后，只进不出总是不太合理的，我们还要考虑一个 off() 方法，必要的时候用它来删除用不到的监听器：

off()：负责监听器的删除。

在之前的发布的博客中有实现过发布订阅模式，这里就不在实现了

## 使用 Context API 维护全局状态

Context API 是 React 官方提供的一种组件树全局通信的方式

![图片](https://img.cdn.sugarat.top/mdImg/MTY0ODE5NjcwMDIzMg==648196700232)

Cosumer 不仅能够读取到 Provider 下发的数据，**还能读取到这些数据后续的更新**。这意味着数据在生产者和消费者之间能够及时同步，这对 Context 这种模式来说至关重要

### 从编码的角度认识“三要素

React.createContext，作用是创建一个 context 对象。下面是一个简单的用法示范：

```react
const AppContext = React.createContext()
```

注意，在创建的过程中，我们可以选择性地传入一个 defaultValue：

```react
const AppContext = React.createContext(defaultValue)
```

从创建出的 context 对象中，我们可以读取到 Provider 和 Consumer：

```react
const { Provider, Consumer } = AppContext
```

Provider，可以理解为“数据的 Provider（提供者）”。

我们使用 Provider 对组件树中的根组件进行包裹，然后传入名为“value”的属性，这个 value 就是后续在组件树中流动的“数据”，它可以被 Consumer 消费。使用示例如下：

```react
<Provider value={title: this.state.title, content: this.state.content}>
   <Title />
  <Content />
 </Provider>
```

Consumer，顾名思义就是“数据的消费者”，它可以读取 Provider 下发下来的数据。

其特点是需要接收一个函数作为子元素，这个函数需要返回一个组件。像这样：

```react
<Consumer>
  {value => <div>{value.title}</div>}
</Consumer>
```

注意，当 Consumer 没有对应的 Provider 时，value 参数会直接取创建 context 时传递给 createContext 的 defaultValue。

## 第三方数据流框架： Redux

对于简单的跨层级组件通信，我们可以使用发布-订阅模式或者 Context API 来搞定。但是随着应用的复杂度不断提升，需要维护的状态越来越多，组件间的关系也越来越难以处理的时候，我们就需要请出 Redux 来帮忙了。

### 什么是 Redux

我们先来看一下官方对 Redux 的描述：

> Redux 是 JavaScript 状态容器，它提供可预测的状态管理。

Redux 是为JavaScript应用而生的，也就是说它不是 React 的专利，React 可以用，Vue 可以用，原生 JavaScript 也可以用；

Redux 是一个状态容器，什么是状态容器？这里我举个例子。

假如把一个 React 项目里面的所有组件拉进一个钉钉群，那么 Redux 就充当了这个群里的“群文件”角色，所有的组件都可以把需要在组件树里流动的数据存储在群文件里。当某个数据改变的时候，其他组件都能够通过下载最新的群文件来获取到数据的最新值。这就是“状态容器”的含义——存放公共数据的仓库。

如下图：

![图片](https://img.cdn.sugarat.top/mdImg/MTY0ODE5NzE2Mzc3Nw==648197163777)

Redux 是如何帮助 React 管理数据的
Redux 主要由三部分组成：store、reducer 和 action。我们先来看看它们各自代表什么：

- store 就好比组件群里的“群文件”，它是一个单一的数据源，而且是只读的；

- action 人如其名，是“动作”的意思，它是对变化的描述。


举个例子，下面这个对象就是一个 action：

```react
const action = {
  type: "ADD_ITEM",
  payload: '<li>text</li>'
}
```

- 
  reducer 是一个函数，它负责**对变化进行分发和处理**， 最终将新的数据返回给 store。

store、action 和 reducer 三者紧密配合，便形成了 Redux 独树一帜的工作流：


从上图中，我们首先读出的是数据的流向规律：在 Redux 的整个工作过程中，数据流是严格单向的。

接下来仍然是围绕上图，我们来一起看看 Redux 是如何帮助 React 管理数据流的。对于一个 React 应用来说，视图（View）层面的所有数据（state）都来自 store（再一次诠释了单一数据源的原则）。

如果你想对数据进行修改，只有一种途径：派发 action。action 会被 reducer 读取，进而根据 action 内容的不同对数据进行修改、生成新的 state（状态），这个新的 state 会更新到 store 对象里，进而驱动视图层面做出对应的改变。

对于组件来说，任何组件都可以通过约定的方式从 store 读取到全局的状态，任何组件也都可以通过合理地派发 action 来修改全局的状态。Redux 通过提供一个统一的状态容器，使得数据能够自由而有序地在任意组件之间穿梭，这就是 Redux 实现组件间通信的思路。

### 从编码的角度理解 Redux 工作流

1. #### 使用 createStore 来完成 store 对象的创建

```react
// 引入 redux
import { createStore } from 'redux'
// 创建 store
const store = createStore(
    reducer,
    initial_state,
    applyMiddleware(middleware1, middleware2, ...)
);
```

createStore 方法是一切的开始，它接收三个入参：

- reducer；

- 初始状态内容；

- 指定中间件。


这其中一般来说，只有 reducer 是你不得不传的。下面我们就看看 reducer 的编码形态是什么样的。

2. #### reducer 的作用是将新的 state 返回给 store

一个 reducer 一定是一个纯函数，它可以有各种各样的内在逻辑，但它最终一定要返回一个 state：

```react
const reducer = (state, action) => {
    // 此处是各种样的 state处理逻辑
    return new_state
}
```

当我们基于某个 reducer 去创建 store 的时候，其实就是给这个 store 指定了一套更新规则：

```react
// 更新规则全都写在 reducer 里 
const store = createStore(reducer)
```

3. action 的作用是通知 reducer “让改变发生”

如何在浩如烟海的 store 状态库中，准确地命中某个我们希望它发生改变的 state 呢？reducer 内部的逻辑虽然不尽相同，但其本质工作都是“将 action 与和它对应的更新动作对应起来，并处理这个更新”。所以说要想让 state 发生改变，就必须用正确的 action 来驱动这个改变。

前面我们已经介绍过 action 的形态，这里再提点一下。首先，action 是一个大致长这样的对象：

```react
const action = {
  type: "ADD_ITEM",
  payload: '<li>text</li>'
}
```

action 对象中允许传入的属性有多个，但只有 type 是必传的。type 是 action 的唯一标识，reducer 正是通过不同的 type 来识别出需要更新的不同的 state，由此才能够实现精准的“定向更新”。

4. 派发 action，靠的是 dispatch

action 本身只是一个对象，要想让 reducer 感知到 action，还需要“派发 action”这个动作，这个动作是由 store.dispatch 完成的。这里我简单地示范一下：



```react
import { createStore } from 'redux'
// 创建 reducer
const reducer = (state, action) => {
    // 此处是各种样的 state处理逻辑
    return new_state
}
// 基于 reducer 创建 state
const store = createStore(reducer)
// 创建一个 action，这个 action 用 “ADD_ITEM” 来标识 
const action = {
  type: "ADD_ITEM",
  payload: '<li>text</li>'
}
// 使用 dispatch 派发 action，action 会进入到 reducer 里触发对应的更新
store.dispatch(action)
```

以上这段代码，是从编码角度对 Redux 主要工作流的概括，总结了一张对应的流程图：

![图片](https://img.cdn.sugarat.top/mdImg/MTY0ODE5ODU0ODU0NQ==648198548545)

