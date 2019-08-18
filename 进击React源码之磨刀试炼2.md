[TOC]
# 进击React源码之磨刀试炼2

进击React源码之磨刀试炼部分为源码解读基础部分，会包含多篇文章，本篇为第二篇，[第一篇入口](https://github.com/kingshuaishuai/blog/blob/master/%E8%BF%9B%E5%87%BBReact%E6%BA%90%E7%A0%81%E4%B9%8B%E7%A3%A8%E5%88%80%E8%AF%95%E7%82%BC1.md)（点击进入）。

## 初探Component与PureComponent

如果有没用过`PureComponent`或不了解的同学，可以看看这篇文章[何时使用Component还是PureComponent？](https://segmentfault.com/a/1190000014979065?utm_source=tag-newest)

### 猜猜组件内部如何实现？
Component（组件）作为React中最重要的概念，每当创建类组件都要继承`Component`或`PureComponent`，在未开始看源码的时候，大家可以先跟自己谈谈对于`Component`和`PureComponent`的印象，不妨根据经验猜一猜`Component`内部将会为我们实现怎样的功能？

先来写个简单的组件

``` jsx
class CompDemo extends PureComponent {
  constructor(props) {
    super(props);
    this.state = {
      msg: 'hello world'
    }
  }

  componentDidMount() {
    setTimeout(() => {
      this.setState({
        msg: 'Hello React'
      });
    }, 1000)
  }

  render() {
    return (
      <div className="CompDemo">
        <div className="CompDemo__text">
          {this.state.msg}
        </div>
      </div>
    )
  }
}
```

![](https://www.github.com/kingshuaishuai/static_resource/raw/master/assets/1566097389258.png)
通过这个简单的组件，我们猜猜，`Component`/`PureComponent`组件内部可能帮我们处理了`props`,`state`,定义了生命周期函数，`setState`，`render`等很多功能。

### 源码实现
打开`packages/react/src/ReactBaseClasses.js`,打开后里面有很多英文注释，希望大家不管通过什么手段先翻译看看，自己先大致了解一下。之后贴出的源码中我会过滤掉自带的注释和`if(__DEV__)`语句，有兴趣了解的同学可以翻阅源码研究。

**Component**

``` javascript
function Component(props, context, updater) {
  this.props = props;
  this.context = context;
  this.refs = emptyObject;
  this.updater = updater || ReactNoopUpdateQueue;
}

Component.prototype.isReactComponent = {};

Component.prototype.setState = function(partialState, callback) {
  invariant(
    typeof partialState === 'object' ||
      typeof partialState === 'function' ||
      partialState == null,
    'setState(...): takes an object of state variables to update or a ' +
      'function which returns an object of state variables.',
  );
  this.updater.enqueueSetState(this, partialState, callback, 'setState');
};

Component.prototype.forceUpdate = function(callback) {
  this.updater.enqueueForceUpdate(this, callback, 'forceUpdate');
};

```
以上就是Component相关的源码，它竟如此出奇的简洁！字面来看懂它也很简单，首先定义了`Component`构造函数，之后在其原型链上设置了`isReactComponent`(Component组件标志)、`setState`方法和`forceUpdate`方法。

`Component`构造函数可以接收三个参数，其中`props`和`context`我们大多数人应该都接触过，在函数中还定义了`this.refs`为一个空对象，但`updater`就是一个比较陌生的东西了，在`setState`和`forceUpdate`方法中我们可以看到它的使用：

* `setState`并没有具体实现更新state的方法，而是调用了`updater`的`enqueueSetState`，`setState`接收两个参数：`partialState`就是我们要更新的`state`内容，`callback`可以让我们在`state`更新后做一些自定义的操作，`this.updater.enqueueSetState`在这里传入了四个参数，我们可以猜到第一个为当前实例对象，第二个是我们更新的内容，第三个是传入的`callback`，最后一个是当前操作的名称。这段代码上面`invariant`的作用是判断`partialState`是否是对象、函数或者`null`，如果不是则会给出提示。在这里我们可以看出，`setState`第一个参数不仅可以为`Object`，也可以是个函数，大家在实际操作中可以尝试使用。
* `forceUpdate`相比于`setState`，只有`callback`，同时在使用`enqueueForceUpdate`时候也少传递了一个参数，其他参数跟`setState`中调用保持一致。

这个`updater.enqueueForceUpdate`来自`ReactDom`，`React`与`ReactDom`是分开的两个不同的内容,很多复杂的操作都被封装在了`ReactDom`中，因此`React`才保持如此简洁。`React`在不同平台（native和web）使用的都是相同的代码，但是不同平台的DOM操作流程可能是不同的,因此将`state`的更新操作通过对象方式传递过来,可以让不同的平台去自定义自己的操作逻辑,`React`就可以专注于大体流程的实现。

**PureComponent**

``` javascript
function ComponentDummy() {}
ComponentDummy.prototype = Component.prototype;

function PureComponent(props, context, updater) {
  this.props = props;
  this.context = context;
  this.refs = emptyObject;
  this.updater = updater || ReactNoopUpdateQueue;
}

const pureComponentPrototype = (PureComponent.prototype = new ComponentDummy());
pureComponentPrototype.constructor = PureComponent;

Object.assign(pureComponentPrototype, Component.prototype);
pureComponentPrototype.isPureReactComponent = true;
```
看完`Cpomponent`的内容，再看`PureComponent`就很简单了，单看`PureComponent`的定义是与`Component`是完全一样的，这里使用了寄生组合继承的方式，让`PureComponent`继承了Component,之后设置了`isPureReactComponent`标志为true。

> 如果有同学对JavaScript继承不是很了解，这里找了一篇掘金上的文章[深入JavaScript继承原理
](https://juejin.im/post/5a96d78ef265da4e9311b4d8#heading-7)大家可以点击进入查看

## Refs的用法与实现
### ref的使用
通过`ref`我们可以获得组件内某个子节点的信息病对其进行操作，ref的使用方式有三种:

``` javascript
class RefDemo extends PureComponent {
  constructor() {
    super()
    this.objRef = React.createRef()
  }

  componentDidMount() {
    setTimeout(() => {
      this.refs.stringRef.textContent = "String ref content changed";
      this.methodRef.textContent = "Method ref content changed";
      this.objRef.current.textContent = "Object ref content changed";
    }, 3000)
  }

  render() {
    return (
      <div className="RefDemo">
        <div className="RefDemo__stringRef" ref="stringRef">this is string ref</div>
        <div className="RefDemo__methodRef" ref={el => this.methodRef = el}>this is method ref</div>
        <div className="RefDemo__objRef" ref={this.objRef}>this is object ref</div>
      </div>
    )
  }
}

export default RefDemo;
```
![Jietu20190818-124212](https://www.github.com/kingshuaishuai/static_resource/raw/master/assets/Jietu20190818-124212.gif)

1. string ref（不推荐，可能废弃）：通过字符串方式设置ref，会在this.refs对象上挂在一个`key`为所设字符串的属性，用来表示该节点的实例对象。如果该节点为dom，则对应dom示例，如果是`class component`则对应该组件实例对象，如果是`function component`，则会出现错误，`function component`没有实例，但可以通过`forward ref`来使用`ref`。
2. method ref：通过function来创建ref（笔者在之前实习工作中基本都是使用这种方式，非常好用）。
3. 通过`createRef()`创建对象，默认创建的对象为`{current: null}`,将其传递个某个节点，在组件渲染结束后会将此节点的实例对象挂在到`current`上

### createRef的实现

源码位置`packages/react/src/ReactCreactRef.js`
``` javascript
export function createRef(): RefObject {
  const refObject = {
    current: null,
  };
  return refObject;
}
```

它上方有段注释`an immutable object with a single mutable value`，告诉我们创建出来的对象具有单个可变值，但是这个对象是不可变的。在其内部跟我们上面说的一样，创建了`{current: null}`并将其返回。
