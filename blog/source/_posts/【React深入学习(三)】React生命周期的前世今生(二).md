---
title: 【React深入学习(三)】React生命周期的前世今生(二)
date: 2022-2-14 12:58:18
tags:
    - React
categories:
    - React源码
keywords: "框架,React"
description: React的深入浅出
cover: https://img.cdn.sugarat.top/mdImg/MTY0NzkxNDI5ODYxOA==647914298618
---

## React16的生命周期

[生命周期流程图](https://projects.wojtekmaj.pl/react-lifecycle-methods-diagram/)

![图片](https://img.cdn.sugarat.top/mdImg/MTY0Nzk0MTI3ODExNg==647941278116)

React 16 以来的生命周期也可以按照“挂载”“更新”和“卸载”三个阶段来看，所以接下来我们要做的事情仍然是分阶段拆解工作流程

我们通过下面这个demo来学习

```react
import React from "react";
import ReactDOM from "react-dom";

// 定义子组件
class LifeCycle extends React.Component {
  constructor(props) {
    console.log("进入constructor");
    super(props);

    // state 可以在 constructor 里初始化
    this.state = { text: "子组件的文本" };
  }

  // 初始化/更新时调用
  static getDerivedStateFromProps(props, state) {
    console.log("getDerivedStateFromProps方法执行");
    return {
      fatherText: props.text
    }
  }

  // 初始化渲染时调用
  componentDidMount() {
    console.log("componentDidMount方法执行");
  }

  // 组件更新时调用
  shouldComponentUpdate(prevProps, nextState) {
    console.log("shouldComponentUpdate方法执行");
    return true;
  }



  // 组件更新时调用
  getSnapshotBeforeUpdate(prevProps, prevState) {
    console.log("getSnapshotBeforeUpdate方法执行");
    return "haha";
  }

  // 组件更新后调用

  componentDidUpdate(preProps, preState, valueFromSnapshot) {
    console.log("componentDidUpdate方法执行");
    console.log("从 getSnapshotBeforeUpdate 获取到的值是", valueFromSnapshot);
  }

  // 组件卸载时调用
  componentWillUnmount() {
    console.log("子组件的componentWillUnmount方法执行");
  }

  // 点击按钮，修改子组件文本内容的方法
  changeText = () => {
    this.setState({
      text: "修改后的子组件文本"
    });
  };

  render() {
    console.log("render方法执行");
    return (
      <div className="container">
        <button onClick={this.changeText} className="changeText">
          修改子组件文本内容
        </button>
        <p className="textContent">{this.state.text}</p>
        <p className="fatherContent">{this.props.text}</p>
      </div>
    );
  }
}

// 定义 LifeCycle 组件的父组件
class LifeCycleContainer extends React.Component {
  // state 也可以像这样用属性声明的形式初始化
  state = {
    text: "父组件的文本",
    hideChild: false
  }

  // 点击按钮，修改父组件文本的方法
  changeText = () => {
    this.setState({
      text: "修改后的父组件文本"
    });
  };

  // 点击按钮，隐藏（卸载）LifeCycle 组件的方法
  hideChild = () => {
    this.setState({
      hideChild: true
    });
  };

  render() {
    return (
      <div className="fatherContainer">
        <button onClick={this.changeText} className="changeText">
          修改父组件文本内容
        </button>
        <button onClick={this.hideChild} className="hideChild">
          隐藏子组件
        </button>
        {this.state.hideChild ? null : <LifeCycle text={this.state.text} />}
      </div>
    );
  }
}

ReactDOM.render(<LifeCycleContainer />, document.getElementById("root"));

```



## Mounting阶段：组件的初始化渲染(挂载)

我们对比一下16与15两个生命周期的差异

![图片](https://img.cdn.sugarat.top/mdImg/MTY0Nzk0MDAyMzQ1Mg==647940023452)

现在我们打开demo可以看见

![图片](https://img.cdn.sugarat.top/mdImg/MTY0Nzk0MDI0ODQzMg==647940248432)

##### 消失的 componentWillMount，新增的 getDerivedStateFromProps

##### 注意：getDerivedStateFromProps 不是 componentWillMount 的替代品

事实上，**componentWillMount 的存在不仅“鸡肋”而且危险，因此它并不值得被“代替”，它就应该被废弃。** 后面会举例说明

而 getDerivedStateFromProps 这个 API，其设计的初衷不是试图替换掉 **componentWillMount**，而是试图替换掉 **componentWillReceiveProps**，因此它有且仅有一个用途：**使用 props 来派生/更新 state**。

React 团队为了确保 getDerivedStateFromProps 这个生命周期的纯洁性，直接从命名层面约束了它的用途（getDerivedStateFromProps 直译过来就是“从 Props 里派生 State”）。所以，如果你不是出于这个目的来使用 getDerivedStateFromProps，原则上来说都是不允许的

值得一提的是，getDerivedStateFromProps 在**更新和挂载**两个阶段都会“出镜”（这点不同于仅在更新阶段出现的 componentWillReceiveProps）。这是因为“派生 state”这种诉求不仅在 props 更新时存在，**在 props 初始化的时候也是存在的**。

### 认识getDerivedStateFromProps(props,state)

我们需要了解三个点

第一个重点是最特别的一点：**getDerivedStateFromProps 是一个静态方法**。静态方法不依赖组件实例而存在，因此你在这个方法内部是**访问不到 this** 的

第二个重点，该方法可以接收两个参数：props 和 state，它们分别代表当前组件接收到的来自父组件的 props 和当前组件自身的 state

![图片](https://img.cdn.sugarat.top/mdImg/MTY0Nzk1NzUzMzc3Nw==647957533777)

第三个重点，getDerivedStateFromProps 需要一个对象格式的返回值。如果你没有指定这个返回值，那么大概率会被 React 警告一番

![图片](https://img.cdn.sugarat.top/mdImg/MTY0Nzk1NzY3OTcwNg==647957679706)

**getDerivedStateFromProps 的返回值之所以不可或缺，是因为 React 需要用这个返回值来更新（派生）组件的 state**。因此当你确实不存在“使用 props 派生 state ”这个需求的时候，最好是直接省略掉这个生命周期方法的编写，否则一定记得给它 return 一个 null

注意：**getDerivedStateFromProps 方法对 state 的更新动作并非“覆盖”式的更新**，**而是针对某个属性的定向更新**。比如这里我们在 getDerivedStateFromProps 里返回的是这样一个对象，对象里面有一个 fatherText 属性用于表示“父组件赋予的文本”：

![图片](https://img.cdn.sugarat.top/mdImg/MTY0Nzk1NzgzNTU3Nw==647957835577)

## Updating 阶段：组件的更新

![图片](https://img.cdn.sugarat.top/mdImg/MTY0Nzk1NzkwMjU1NA==647957902554)

#### 为什么要用 getDerivedStateFromProps 代替 componentWillReceiveProps？

- getDerivedStateFromProps 是作为一个**试图代替 componentWillReceiveProps** 的 API 而出现的；
- getDerivedStateFromProps**不能完全和 componentWillReceiveProps 画等号**，其特性决定了我们曾经在 componentWillReceiveProps 里面做的事情，不能够百分百迁移到getDerivedStateFromProps 里

从 getDerivedStateFromProps 直接被定义为 static 方法这件事上就可见一斑—— static 方法内部拿不到组件实例的 this，这就导致你无法在 getDerivedStateFromProps 里面做任何类似于 this.fetch()、不合理的 this.setState（会导致死循环的那种）这类可能会产生副作用的操作，最终的答案就是为了**避免不正确的操作**

因此，getDerivedStateFromProps 生命周期替代 componentWillReceiveProps 的背后，**是 React 16 在强制推行“只用 getDerivedStateFromProps 来完成 props 到 state 的映射”这一最佳实践**。意在确保生命周期函数的行为更加可控可预测，从根源上帮开发者避免不合理的编程方式，避免生命周期的滥用；同时，也是在为新的 Fiber 架构铺路

#### 消失的 componentWillUpdate 与新增的 getSnapshotBeforeUpdate

getSnapshotBeforeUpdate 

```js
getSnapshotBeforeUpdate(prevProps, prevState) {
  // ...
}
```

这个方法和 getDerivedStateFromProps 颇有几分神似，它们都强调了需要一个返回值。区别在于 **getSnapshotBeforeUpdate 的返回值会作为第三个参数给到 componentDidUpdate**。**它的执行时机是在 render 方法之后，真实 DOM 更新之前**。在这个阶段里，我们可以**同时获取到更新前的真实 DOM 和更新前后的 state&props 信息**。

尽管在实际工作中，需要用到这么多信息的场景并不多，但在对于实现一些特殊的需求来说，没它还真的挺难办。这里我举一个非常有代表性的例子：实现一个内容会发生变化的滚动列表，要求根据滚动列表的内容是否发生变化，来决定是否要记录滚动条的当前位置。

这个需求的前半截要求我们对比更新前后的数据（感知变化），后半截则需要获取真实的 DOM 信息（获取位置），这时用 getSnapshotBeforeUpdate 来解决就再合适不过了。

#### getSnapshotBeforeUpdate与componentDidUpdate的通信

```react
// 组件更新时调用
getSnapshotBeforeUpdate(prevProps, prevState) {
  console.log("getSnapshotBeforeUpdate方法执行");
  return "haha";
}

// 组件更新后调用
componentDidUpdate(prevProps, prevState, valueFromSnapshot) {
  console.log("componentDidUpdate方法执行");
  console.log("从 getSnapshotBeforeUpdate 获取到的值是", valueFromSnapshot);
}
```

![图片](https://img.cdn.sugarat.top/mdImg/MTY0Nzk1ODY2MjU0OA==647958662548)

值得一提的是，这个生命周期的设计初衷，是为了“与 componentDidUpdate 一起，涵盖过时的 componentWillUpdate 的所有用例”（引用自 React 官网）。**getSnapshotBeforeUpdate 要想发挥作用，离不开 componentDidUpdate 的配合**。

那么换个角度想想，**为什么 componentWillUpdate 就非死不可呢**？说到底，还是因为它“挡了 Fiber 的路”。

## Unmounting 阶段：组件的卸载

卸载阶段的生命周期与 React 15 完全一致，只涉及 componentWillUnmount 这一个生命周期

![图片](https://img.cdn.sugarat.top/mdImg/MTY0Nzk1OTE2MjU1Mw==647959162553)

## 总结：React15到16的缘何

在 React 16 之前，每当我们触发一次组件的更新，React 都会构建一棵新的虚拟 DOM 树，通过与上一次的虚拟 DOM 树进行 diff，实现对 DOM 的定向更新。这个过程，是一个递归的过程。下面这张图形象地展示了这个过程的特征：

![图片](https://img.cdn.sugarat.top/mdImg/MTY0ODA5MTI2NjE0MA==648091266140)

同步渲染的递归调用栈是非常深的，只有最底层的调用返回了，整个渲染过程才会开始逐层返回。这个漫长且不可打断的更新过程，将会带来用户体验层面的巨大风险：同步渲染一旦开始，便会牢牢抓住主线程不放，直到递归彻底完成。在这个过程中，浏览器没有办法处理任何渲染之外的事情，会进入一种无法处理用户交互的状态。因此若渲染时间稍微长一点，页面就会面临卡顿甚至卡死的风险。

而 React 16 引入的 Fiber 架构，恰好能够解决掉这个风险：**Fiber 会将一个大的更新任务拆解为许多个小任务**。每当执行完一个小任务时，**渲染线程都会把主线程交回去**，看看有没有优先级更高的工作要处理，确保不会出现其他任务被“饿死”的情况，进而避免同步渲染带来的卡顿。在这个过程中，渲染线程不再“一去不回头”，而是可以被打断的，这就是所谓的“异步渲染”，它的执行过程如下图所示：

![图片](https://img.cdn.sugarat.top/mdImg/MTY0ODA5MTMzMjIyNQ==648091332225)

### 那么从这个角度来看生命周期

Fiber 架构的重要特征就是可以被打断的异步渲染模式。但这个“打断”是有原则的，根据“能否被打断”这一标准，React 16 的生命周期被划分为了 render 和 commit 两个阶段，而 commit 阶段又被细分为了 pre-commit 和 commit。每个阶段所涵盖的生命周期如下图所示

![图片](https://img.cdn.sugarat.top/mdImg/MTY0ODEwNTQ3NjMyMw==648105476323)

- render 阶段：纯净且没有副作用，可能会被 React 暂停、终止或重新启动。

- pre-commit 阶段：可以读取 DOM。

- commit 阶段：可以使用 DOM，运行副作用，安排更新。

**总的来说，render 阶段在执行过程中允许被打断，而 commit 阶段则总是同步执行的**

简单来说，由于 render 阶段的操作对用户来说其实是“不可见”的，所以就算打断再重启，对用户来说也是零感知。而 commit 阶段的操作则涉及真实 DOM 的渲染，再狂的框架也不敢在用户眼皮子底下胡乱更改视图，所以这个过程必须用同步渲染来求稳

### 细说生命周期“废旧立新”背后

在 Fiber 机制下，**render 阶段是允许暂停、终止和重启**的。当一个任务执行到一半被打断后，下一次渲染线程抢回主动权时，这个任务被重启的形式是“重复执行一遍整个任务”而非“接着上次执行到的那行代码往下走”。这就导致 render 阶段的生命周期都是有可能被重复执行的。

带着这个结论，我们再来看看 React 16 打算废弃的是哪些生命周期：

componentWillMount；

componentWillUpdate；

componentWillReceiveProps。

这些生命周期的共性，就是它们都处于 render 阶段，都可能重复被执行，它们在重复执行的过程中都存在着不可小觑的风险。

setState()；

fetch 发起异步请求；

操作真实 DOM。

这些操作的问题（或不必要性）包括但不限于以下 3 点：

（1）完全可以转移到其他生命周期（尤其是 componentDidxxx）里去做。

比如在 componentWillMount 里发起异步请求。很多同学因为太年轻，以为这样做就可以让异步请求回来得“早一点”，从而避免首次渲染白屏。

可惜你忘了，异步请求再怎么快也快不过（React 15 下）同步的生命周期。componentWillMount 结束后，render 会迅速地被触发，所以说首次渲染依然会在数据返回之前执行。这样做不仅没有达到你预想的目的，还会导致服务端渲染场景下的冗余请求等额外问题，得不偿失。

（2）在 Fiber 带来的异步渲染机制下，可能会导致非常严重的 Bug。

试想，假如你在 componentWillxxx 里发起了一个付款请求。由于 render 阶段里的生命周期都可以重复执行，在 componentWillxxx 被打断 + 重启多次后，就会发出多个付款请求。

比如说，这件商品单价只要 10 块钱，用户也只点击了一次付款。但实际却可能因为 componentWillxxx 被打断 + 重启多次而多次调用付款接口，最终付了 50 块钱；又或者你可能会习惯在 componentWillReceiveProps 里操作 DOM（比如说删除符合某个特征的元素），那么 componentWillReceiveProps 若是执行了两次，你可能就会一口气删掉两个符合该特征的元素。

结合上面的分析，我们再去思考 getDerivedStateFromProps 为何会在设计层面直接被约束为一个触碰不到 this 的静态方法，其背后的原因也就更加充分了——**避免开发者触碰 this，就是在避免各种危险的骚操作。**

（3）即使你没有开启异步，React 15 下也有不少人能把自己“玩死”。

总的来说，React 16 改造生命周期的主要动机是为了配合 Fiber 架构带来的异步渲染机制。这一系列的工作做下来，首先是确保了 Fiber 机制下数据和视图的安全性，同时也确保了生命周期方法的行为更加纯粹、可控、可预测。
