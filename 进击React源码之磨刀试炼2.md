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

**forwardRef的使用**

``` javascript
const FunctionComp = React.forwardRef((props, ref) => (
  <div type="text" ref={ref}>Hello React</div>
))

class FnRefDemo extends PureComponent {
  constructor() {
    super();
    this.ref = React.createRef();
  }

  componentDidMount() {
    setTimeout(() => {
      this.ref.current.textContent = "Changed"
    }, 3000)
  }

  render() {
    return (
      <div className="RefDemo">
        <FunctionComp ref={this.ref}/>
      </div>
    )
  }
}
```

![](https://raw.githubusercontent.com/kingshuaishuai/static_resource/master/assets/Jietu20190818-131824.gif)

`forwardRef`的使用，可以让`Function Component`使用ref，传递参数时需要注意传入第二个参数`ref`

**forwardRef的实现**

``` javascript
export default function forwardRef<Props, ElementType: React$ElementType>(
  render: (props: Props, ref: React$Ref<ElementType>) => React$Node,
) {
  return {
    $$typeof: REACT_FORWARD_REF_TYPE,
    render,
  };
}

```
`forwardRef`接收一个函数作为参数，这个函数就是我们的函数组件，它包含`props`和`ref`属性，`forwardRef`最终返回的是一个对象，这个对象包含两个属性：
1. `$$typeof`：这个属性看过上[一篇文章](https://github.com/kingshuaishuai/blog/blob/master/%E8%BF%9B%E5%87%BBReact%E6%BA%90%E7%A0%81%E4%B9%8B%E7%A3%A8%E5%88%80%E8%AF%95%E7%82%BC1.md)的小伙伴应该还记得，它是标志React Element类型的东西。
2. render: 我们传递进来的函数组件。

这里说明一下，尽管`forwardRef`返回的对象中`$$typeof`为`REACT_FORWARD_REF_TYPE`,但是最终创建的ReactElement的$$typeof仍然是`REACT_ELEMENT_TYPE`

这里文字描述有点绕，配合图片来看文字会好点。
![enter description here](https://www.github.com/kingshuaishuai/static_resource/raw/master/assets/1566108414500.png)

在上述`forwardRef使用`的代码中创建的`FunctionComp`是`{$$typeof:REACT_FORWARD_REF_TYPE,render}`这个对象，在使用`<FunctionComp ref={this.ref}/>`时，它的本质是`React.createElement(FunctionComp, {ref: xxxx}, null)`这样的，此时`FunctionComp`是我们传进`createElement`中的`type`参数，`createElement`返回的`element`的`$$typeof`仍然是`REACT_ELEMENT_TYPE`；

## ReactChildren的使用方法和实现
### ReactChildren的使用

``` jsx
function ParentComp ({children}) {
  return (
    <div className="parent">
      <div className="title">Parent Component</div>
      <div className="content">
        {children}
      </div>
    </div>
  )
}
```
这样的代码大家平时用的应该多一点，在使用`ParentComp`组件时候，可以在标签中间写一些内容，这些内容就是children。

**来看看React.Children.map的使用**

``` javascript
function ParentComp ({children}) {
  return (
    <div className="parent">
      <div className="title">Parent Component</div>
      <div className="content">
        {React.Children.map(children, c => [c,c, [c]])}
      </div>
    </div>
  )
}

class ChildrenDemo extends PureComponent{
  constructor() {
    super()
    this.state = {}
  }

  render() {
    return (
      <div className="childrenDemo">
        <ParentComp>
          <div>child 1 content</div>
          <div>child 2 content</div>
          <div>child 3 content</div>
        </ParentComp>
      </div>
    )
  }
}

export default ChildrenDemo;
```
![结果](https://www.github.com/kingshuaishuai/static_resource/raw/master/assets/1566312728078.png)

我们在使用这个API的时候，传递了两个参数，第一个是`children`，大家应该比较熟悉，第二个是一个回调函数，回调函数传入一个参数（代表children的一个元素），返回一个数组（数组不是一位数组，里面三个元素最后一个还是数组），在结果中我们可以看到，这个API将我们返回的数组平铺为一层[c1,c1,c1,c2,c2,c2,c3,c3,c3]，浏览器中显示的也就如上图所示。

> 有兴趣的小伙伴可以尝试阅读[官方文档](https://reactjs.org/docs/react-api.html#reactchildrenmap)对于这个api的介绍

### ReactChildren的实现
在`react.js`中定义`React`时候我们可以看到一段关于`Children`的定义

``` javascript
  Children: {
    map,
    forEach,
    count,
    toArray,
    only,
  },
```
Children包含5个API，这里我们先详细讨论map API。这一部分并不是很好懂，请大家看的时候一定要用心。

笔者读这一部分也是费了很大的劲，然后用思维导图软件画出了这个思维导图+流程图的东西（暂时就给它起名为思维流程图，其实更流程一点，而不思维），画得还是比较详细的，所以就很大，小伙伴最好把这个图下载下来放大看（可以配合源码，也可以配合下文），图片地址[https://www.github.com/kingshuaishuai/static_resource/raw/master/assets/1566311843654.png](https://www.github.com/kingshuaishuai/static_resource/raw/master/assets/1566311843654.png)


![enter description here](https://www.github.com/kingshuaishuai/static_resource/raw/master/assets/1566311843654.png)

由于图太小不清楚，下面也会分别截出每个函数的流程图。

打开`packages/react/src/ReactChildren.js`，找到`mapChildren`

``` javascript
function mapChildren(children, func, context) {
  if (children == null) {
    return children;
  }
  const result = [];
  mapIntoWithKeyPrefixInternal(children, result, null, func, context);
  return result;
}
```

![enter description here](https://www.github.com/kingshuaishuai/static_resource/raw/master/assets/1566314553816.png)

这段代码短小精悍，给我们提供了直接使用的API。它内部逻辑也非常简单，首先看看`children`是否为`null`,如果如果为`null`就直接返回`null`，如果不是，则定义`result`（初始为空数组）来存放结果，经过`mapIntoWithKeyPrefixInternal`的一系列处理，得到结果。结果不管是`null`还是`result`，其实我们再写代码的时候都遇到过，如果一个组件中间什么都没传，结果就是null什么都不会显示，如果传递了一个`<div>`那就显示这个`div`，如果传递了一组`div`那就显示这一组（此时就是children不为null的情况），最后显示出来的东西也就是`result`这个数组。

**这一系列处理就是什么处理？**

``` javascript
function mapIntoWithKeyPrefixInternal(children, array, prefix, func, context) {
  let escapedPrefix = '';
  if (prefix != null) {
    escapedPrefix = escapeUserProvidedKey(prefix) + '/';
  }
  const traverseContext = getPooledTraverseContext(
    array,
    escapedPrefix,
    func,
    context,
  );
  traverseAllChildren(children, mapSingleChildIntoContext, traverseContext);
  releaseTraverseContext(traverseContext);
}
```
在进入这个函数的时候，一定要注意使用这个函数时候传递进来的参数究竟是哪几个，不然后面传递次数稍微一多就会晕头转向。
![enter description here](https://www.github.com/kingshuaishuai/static_resource/raw/master/assets/1566314430721.png)

从上一个函数跳过来的时候传递了5个参数，大家可以注意一下这五个参数代表的是什么：
1. `children`：我们再组件中间写的JSX代码
2. `result`: 最终处理完成存放结果的数组
3. `prefix`: 前缀，这里为null
4. `func`: 我们在演示使用的过程中传入的第二个参数，是个回调函数`c => [c,c,[c]]`
5. `context`: 上下文对象

这个函数首先对`prefix`前缀字符串做了个处理，处理完之后还是个字符串。然后通过`getPooledTraverseContext`函数从`对象重用池`中拿出一个对象，说到这里，我们就不得不打断一下这个函数的讲解，突然出现一个`对象重用池`的概念，很多人会很懵逼，并且如果强制把这个函数解析完再继续下一个，会让很多读者产生很多疑惑，不利于后面源码的理解。

**暂时跳到`getPooledTraverseContext`看看对象重用池**

``` javascript
const POOL_SIZE = 10;
const traverseContextPool = [];
function getPooledTraverseContext(
  mapResult,
  keyPrefix,
  mapFunction,
  mapContext,
) {
  if (traverseContextPool.length) {
    const traverseContext = traverseContextPool.pop();
    traverseContext.result = mapResult;
    traverseContext.keyPrefix = keyPrefix;
    traverseContext.func = mapFunction;
    traverseContext.context = mapContext;
    traverseContext.count = 0;
    return traverseContext;
  } else {
    return {
      result: mapResult,
      keyPrefix: keyPrefix,
      func: mapFunction,
      context: mapContext,
      count: 0,
    };
  }
}
```

![enter description here](https://www.github.com/kingshuaishuai/static_resource/raw/master/assets/1566315243261.png)

首先看在使用`getPooledTraverseContext`获取对象的时候，传递了4个参数：
1. `array`: 上个函数中对应的`result`，表示最终返回结果的数组
2. `escapedPrefix`: 前缀，一个字符串，没什么好说的
3. `func`: 我们使用API传递的回调函数 `c=>[c,c,[c]]`
4. `context`: 上下文对象

然后我们看看它做了什么，它去一个`traverseContextPool`数组（这个数组默认为空数组，最多存放10个元素）中尝试`pop`取出一个元素，如果能取出来的话，这个元素是一个对象，有5个属性，这里会把传进来的4个参数保存在这四个元素中，方便后面使用，另外一个属性是个用来计数的计数器。如果没取出来，就返回一个新对象，包含的也是这五个属性。这里要跟大家说说`对象重用池`了。这个对象有5个属性，如果每次使用这个对象都重新创建一个，那么会有较大的创建对象开销，为了节省这部分创建的开销，我们可以在使用完这个对象之后，把它的5个属性都置为空(count就是0了),然后扔回这个数组（`对象重用池`）中，后面要用的时候就直接从`对象重用池`中拿出来，不必重新创建对象，增加开销了。

**再回到`mapIntoWithKeyPrefixInternal`函数中继续向下读**
通过上一步拿到一个带有5个属性的对象之后，继续经过`traverseAllChildren`函数的一系列处理，得到了最终的结果`result`，其中具体内容太多下面再说，然后通过`releaseTraverseContext`函数释放了那个带5个参数的对象。我们先来看看如何释放的：

``` javascript
function releaseTraverseContext(traverseContext) {
  traverseContext.result = null;
  traverseContext.keyPrefix = null;
  traverseContext.func = null;
  traverseContext.context = null;
  traverseContext.count = 0;
  if (traverseContextPool.length < POOL_SIZE) {
    traverseContextPool.push(traverseContext);
  }
}
```
这里也跟我们上面说的`对象重用池有所对应`，这里先把这个对象的5个属性清空，然后看看对象重用池是不是有空，有空的话就把这个清空的属性放进去，方便下次使用，节省创建开销。

**traverseAllChildren和traverseAllChildrenImpl的实现**

``` javascript
function traverseAllChildren(children, callback, traverseContext) {
  if (children == null) {
    return 0;
  }

  return traverseAllChildrenImpl(children, '', callback, traverseContext);
}
```

![enter description here](https://www.github.com/kingshuaishuai/static_resource/raw/master/assets/1566316803465.png)

这个函数基本没做什么重要的事，仅仅判断了`children`是否为`null`，如果是的话就返回0，不是的话就进行具体的处理。还是强调这里传递的参数，一定要注意，看图就可以了，就不用文字描述了。

重要的是`traverseAllChildrenImpl`函数，这个函数有点长，这里给大家分成了两部分，可以分开看

``` javascript
function traverseAllChildrenImpl(
  children,
  nameSoFar,
  callback,
  traverseContext,
) {
// 第一部分
  const type = typeof children;

  if (type === 'undefined' || type === 'boolean') {
    children = null;
  }

  let invokeCallback = false;

  if (children === null) {
    invokeCallback = true;
  } else {
    switch (type) {
      case 'string':
      case 'number':
        invokeCallback = true;
        break;
      case 'object':
        switch (children.$$typeof) {
          case REACT_ELEMENT_TYPE:
          case REACT_PORTAL_TYPE:
            invokeCallback = true;
        }
    }
  }

  if (invokeCallback) {
    callback(
      traverseContext,
      children,
      nameSoFar === '' ? SEPARATOR + getComponentKey(children, 0) : nameSoFar,
    );
    return 1;
  }
  
  // 第二部分

  let child;
  let nextName;
  let subtreeCount = 0; // Count of children found in the current subtree.
  const nextNamePrefix =
    nameSoFar === '' ? SEPARATOR : nameSoFar + SUBSEPARATOR;

  if (Array.isArray(children)) {
    for (let i = 0; i < children.length; i++) {
      child = children[i];
      nextName = nextNamePrefix + getComponentKey(child, i);
      subtreeCount += traverseAllChildrenImpl(
        child,
        nextName,
        callback,
        traverseContext,
      );
    }
  } else {
    const iteratorFn = getIteratorFn(children);
    if (typeof iteratorFn === 'function') {
      const iterator = iteratorFn.call(children);
      let step;
      let ii = 0;
      while (!(step = iterator.next()).done) {
        child = step.value;
        nextName = nextNamePrefix + getComponentKey(child, ii++);
        subtreeCount += traverseAllChildrenImpl(
          child,
          nextName,
          callback,
          traverseContext,
        );
      }
    } else if (type === 'object') {
      let addendum = '';
      const childrenString = '' + children;
      invariant(
        false,
        'Objects are not valid as a React child (found: %s).%s',
        childrenString === '[object Object]'
          ? 'object with keys {' + Object.keys(children).join(', ') + '}'
          : childrenString,
        addendum,
      );
    }
  }

  return subtreeCount;
}
```
![enter description here](https://www.github.com/kingshuaishuai/static_resource/raw/master/assets/1566317158229.png)

上面的流程图说的很详细了，大家可以参照来看源码。这里就简单说一下这个函数的两部分分别作了什么事。
第一部分是对`children`类型进行了检查（没有检查为Array或迭代器对象的情况），如果检查children是合法的ReactElement就会进行`callback`的调用，这里一定要注意`callback`传进来的是谁，这里是`callback为mapSingleChildIntoContext`,一直让大家关注传参问题，就是怕大家看着看着就搞混了。
第二部分就是针对`children`是数组和迭代器对象的情况进行了处理（迭代器对象检查的原理是`obj[Symbol.iterator]`，比较简单大家可以自己定位源码找一下具体实现），然后对他们进行遍历，每个元素都重新执行`traverseAllChildrenImpl`函数形成递归。
它其实只让可渲染的单元素进行下一步`callback`的调用，如果是数组或迭代器，就进行遍历。

**最后一步callback => mapSingleChildIntoContext的实现**

``` javascript
function mapSingleChildIntoContext(bookKeeping, child, childKey) {
  const {result, keyPrefix, func, context} = bookKeeping;

  let mappedChild = func.call(context, child, bookKeeping.count++);
  if (Array.isArray(mappedChild)) {
    mapIntoWithKeyPrefixInternal(mappedChild, result, childKey, c => c);
  } else if (mappedChild != null) {
    if (isValidElement(mappedChild)) {
      mappedChild = cloneAndReplaceKey(
        mappedChild,
        // Keep both the (mapped) and old keys if they differ, just as
        // traverseAllChildren used to do for objects as children
        keyPrefix +
          (mappedChild.key && (!child || child.key !== mappedChild.key)
            ? escapeUserProvidedKey(mappedChild.key) + '/'
            : '') +
          childKey,
      );
    }
    result.push(mappedChild);
  }
}
```
![enter description here](https://www.github.com/kingshuaishuai/static_resource/raw/master/assets/1566318050842.png)

这里我们就用到了从`对象重用池拿出来的对象`，那个对象作用其实就是利用那5个属性帮我们保存了一些需要使用的变量和函数，然后执行我们传入的`func`（`c => [c,c,[c]]`）,如果结果不是数组而是元素并且不为`null`就会直接存储到`result`结果中，如果是个数组就会对它进行遍历，从`mapIntoWithKeyPrefixInternal`开始重新执行形成递归调用，直到最后将嵌套数组中所有元素都拿出来放到`result`中，这样就形成了我们最初看到的那种效果，不管我们的回调函数是多少层的数组，最后都会变成一层。

### 小结
这里文字性的小结就留给大家，给大家画了一张总结性的流程图（有参考yck大神的图），但其实是根据自己看源码画出来的并不是搬运的。
![enter description here](https://www.github.com/kingshuaishuai/static_resource/raw/master/assets/1566311644569.png)

## ReactChildren的其他方法

``` javascript
{
  forEach,
  count,
  toArray,
  only,
}
```
对于这几个方法，大家可以自行查看了，建议先浏览一遍`forEach`，跟`map`非常相似，但是比`map`少了点东西。其他几个都是四五行的代码，大家自己看看。里面用到的函数我们上面都有讲到。

## 小结
这篇文章跟大家一起读了`Component`、`refs`和`Children`相关的源码，最复杂的还是数`Children`了，说实话，连看大神博客，看源码、画图带写文章，花了七八个小时，其实内容跟大神们的文章比起来还是很不一样的，如果基础不是很好的同学，我感觉这里会讲的更详细。
