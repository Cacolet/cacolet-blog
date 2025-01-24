---

title: 【React深入学习(二)】React生命周期的前世今生(一)
date: 2022-2-12 12:58:18
tags:
    - React
categories:
    - React源码
keywords: "框架,React"
description: React的深入浅出
cover: https://img.cdn.sugarat.top/mdImg/MTY0NzkxNDI5ODYxOA==647914298618

---

## 拆解 React 生命周期：从 React 15 说起

关注以下几个生命周期方法

```js
constructor()
componentWillReceiveProps()
shouldComponentUpdate()
componentWillMount()
componentWillUpdate()
componentDidUpdate()
componentDidMount()
render()
componentWillUnmount()
```

他们之间的串联、依存关系：

![图片](https://img.cdn.sugarat.top/mdImg/MTY0Nzg2NTI4MTMwMw==647865281303)

接下来我们来验证一下生命周期的执行顺序，下面有一个demo

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

  // 初始化渲染时调用
  componentWillMount() {
    console.log("componentWillMount方法执行");
  }

  // 初始化渲染时调用
  componentDidMount() {
    console.log("componentDidMount方法执行");
  }

  // 父组件修改组件的props时会调用
  componentWillReceiveProps(nextProps) {
    console.log(nextProps)
    console.log("componentWillReceiveProps方法执行");
  }

  // 组件更新时调用
  shouldComponentUpdate(nextProps, nextState) {
    console.log("shouldComponentUpdate方法执行");
    return true;
  }

  // 组件更新时调用
  componentWillUpdate(nextProps, nextState) {
    console.log("componentWillUpdate方法执行");
  }
  // 组件更新后调用
  componentDidUpdate(preProps, preState) {
    console.log("componentDidUpdate方法执行");
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
  };
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

这是demo的样子

![图片](https://img.cdn.sugarat.top/mdImg/MTY0Nzg3MDUzODI0OQ==647870538249)

### Mounting阶段：组件的初始化渲染(挂载)

挂载阶段流程图

![图片](https://img.cdn.sugarat.top/mdImg/MTY0Nzg2NTc3NDU2OQ==647865774569)

打开控制台可以看见

![图片](https://img.cdn.sugarat.top/mdImg/MTY0Nzg3MDYzMjcxNA==647870632714)

首先我们来看 constructor 方法，该方法仅仅在挂载的时候被调用一次，我们可以在该方法中对 this.state 进行初始化

```react
constructor(props) {
  console.log("进入constructor");
  super(props);
  // state 可以在 constructor 里初始化
  this.state = { text: "子组件的文本" };
}
```

componentWillMount、componentDidMount 方法同样只会在挂载阶段被调用一次。其中 componentWillMount 会在执行 render 方法前被触发，一些同学习惯在这个方法里做一些初始化的操作，但这些操作往往会伴随一些风险或者说不必要性，后面再说

接下来 render 方法被触发。注意 render 在执行过程中并不会去操作真实 DOM（也就是说不会渲染），它的职能是**把需要渲染的内容返回出来**。真实 DOM 的渲染工作，在挂载阶段是由 **ReactDOM.render** 来承接的。

**componentDidMount** 方法在渲染结束后被触发，此时因为真实 DOM 已经挂载到了页面上，我们可以在这个生命周期里执行真实 DOM 相关的操作。此外，类似于异步请求、数据初始化这样的操作也大可以放在这个生命周期来做（侧面印证了 componentWillMount 真的很鸡肋）

### Updating阶段：组件的更新

流程图如下

![图片](https://img.cdn.sugarat.top/mdImg/MTY0Nzg3MDY4MDM1Mg==647870680352)

**componentWillReceiProps 到底是由什么触发的？**

```react
componentWillReceiveProps(nextProps)
```

在这个生命周期方法里，nextProps 表示的是接收到新 props 内容，而现有的 props （相对于 nextProps 的“旧 props”）我们可以通过 this.props 拿到,下面我们会打印出nextProps的内容，可以看到具体的值

经常听到这样的说法**componentWillReceiveProps 是在组件的 props 内容发生了变化时被触发的。**

**这种说法不够严谨**，我们在上面的demo里面改一下，改之前我们先去看一下当我们改变父组件的值的时候

![图片](https://img.cdn.sugarat.top/mdImg/MTY0Nzg3MTY4Njc5NA==647871686794)

我们打印的nextProps值为第一行，然后执行componentWillReceiveProps,后面就是正常的更新流程，


所以目前我们知道componentWillReceiveProps执行时机是在父组件的state更改后传递给子组件的值发生了变化而导致更新的，但真的是这样？

如果我们在父组件中增加一个与子组件毫无关系的值，然后修改这个值，我们看看会不会更新子组件。

```react
// 定义 LifeCycle 组件的父组件

class LifeCycleContainer extends React.Component {
  // state 也可以像这样用属性声明的形式初始化
  state = {
    text: "父组件的文本",
    // 新增的只与父组件有关的 state
    ownText: "仅仅和父组件有关的文本",
    hideChild: false
  };

  changeText = () => {
    this.setState({
      text: "修改后的父组件文本"
    });
  };

  // 修改 ownText 的方法
  changeOwnText = () => {
    this.setState({
      ownText: "修改后的父组件自有文本"
    });
  };

  hideChild = () => {
    this.setState({
      hideChild: true
    });
  };
  render() {
    return (
      <div className="fatherContainer">
        {/* 新的button按钮 */}
        <button onClick={this.changeOwnText} className="changeText">
          修改父组件自有文本内容
        </button>
        <button onClick={this.changeText} className="changeText">
          修改父组件文本内容
        </button>
        <button onClick={this.hideChild} className="hideChild">
          隐藏子组件
        </button>
        <p> {this.state.ownText} </p>
        {this.state.hideChild ? null : <LifeCycle text={this.state.text} />}
      </div>
    );
  }
}
```

demo界面

![图片](https://img.cdn.sugarat.top/mdImg/MTY0Nzg3MjEzMzcxNQ==647872133715)

我们点击修改父组件的自有内容

![图片](https://img.cdn.sugarat.top/mdImg/MTY0Nzg3MjI4NDExOQ==647872284119)

this.state.ownText 这个状态和子组件完全无关。但是当我点击“修改父组件自有文本内容”这个按钮的时候，componentReceiveProps 仍然被触发了，所以**componentReceiveProps 并不是由 props 的变化触发的，而是由父组件的更新触发的**，这个结论，请你谨记。

### 组件自身 setState 触发的更新

this.setState() 调用后导致的更新流程，前面大图中已经有体现，这里我直接沿用上一个 Demo 来做演示。若我们点击Demo 中的“修改子组件文本内容”这个按钮

![图片](https://img.cdn.sugarat.top/mdImg/MTY0Nzg3MjUwMDc4Ng==647872500786)

componentWillUpdate 会在 render 前被触发，它和 componentWillMount 类似，允许你在里面做一些不涉及真实 DOM 操作的准备工作；而 componentDidUpdate 则在组件更新完毕后被触发，和 componentDidMount 类似，这个生命周期也经常被用来处理 DOM 操作。此外，我们也常常将 componentDidUpdate 的执行作为子组件更新完毕的标志通知到父组件

### render与性能：初识shouldComponentUpdate

render 方法由于伴随着对虚拟 DOM 的构建和对比，过程可以说相当耗时。而在 React 当中，很多时候我们会不经意间就频繁地调用了 render。为了避免不必要的 render 操作带来的性能开销，React 为我们提供了 shouldComponentUpdate 这个口子。

React 组件会根据 shouldComponentUpdate 的返回值，来决定是否执行该方法之后的生命周期，进而决定是否对组件进行**re-render**（重渲染）。shouldComponentUpdate 的默认值为 true，也就是说“无条件 re-render”。

### Unmounting 阶段：组件的卸载

![图片](https://img.cdn.sugarat.top/mdImg/MTY0Nzg3Mjc2MzUwNw==647872763507)

这个生命周期本身不难理解，我们重点说说怎么触发它。组件销毁的常见原因有以下两个。

- 组件在父组件中被移除了：这种情况相对比较直观，对应的就是我们上图描述的这个过程。
- 组件中设置了 key 属性，父组件在 render 的过程中，发现 key 值和上一次不一致，那么这个组件就会被干掉。

在这里，只要能够理解到 1 就可以了。对于 2 这种情况，你只需要先记住有这样一种现象，这就够了。至于组件里面为什么要设置 key，为什么 key 改变后组件就必须被干掉？后面再说