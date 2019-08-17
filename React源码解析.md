# 进击React源码之磨刀试炼
## 前言
> 前端的脚步飞快，但是我们实际开发中真正用的到有技术含量的东西真的不是很多，很多时候我们不得不纠结于样式的实现，浪费了大量的时间，也看过很多文章和视频，大佬们一致的结论还是：后端更“编程”，越接近数据的人才会越“值钱”。前端也想做那个“值钱”的人，所以我们就得接触更有技术含量的东西，锻炼真正的编程（而不是排版）能力，并努力拥抱后端。阅读和理解一些开源项目源码便是一种很好的学习和锻炼方式，只不过说起来简单，做起来并不容易，毕竟神仙们的代码，我们不一定看得懂。


>然而幸运的我们可能并不用完全靠自己摸索去读代码，慕课网讲师`jocky`和掘金活跃作者`yck`都在提供带领我们阅读源码的课程/博客,我们可以抓住机会，利用这些优质资源，进行学习和笔记。我也很久没写笔记了，在学习过程中，我的体验还是做笔记会学的更牢固。本系列博客也是基于两位大神的分享与自己的理解展开的。

**两位大神资料地址**
* jocky大神博客(免费)：[https://react.jokcy.me/](https://react.jokcy.me/)
* jocky大神配套慕课视频(收费)：[React源码深度解析-imooc(自愿分享非广告和拖)](https://coding.imooc.com/class/309.html)
* yck大神博客(免费)： [https://yuchengkai.cn/react/](https://yuchengkai.cn/react/)

## 准备工作
clone一份yck大神带注释的源码(版本16.8.6)或者去imooc购买课程可以看jocky大神课程对应的源码（当然感觉没时间看视频或者经费紧张的小伙伴可以先研究博客，自己感觉这样下来学到东西会更多），当然自己也得有一份官方的源码，这里我clone了16.9.0的。
>在我下载的时候速度只有几k，最高不过15k，以前都还是好的，如果遇到类似问题，请各位百度搜索github下载速度慢相关问题寻找解决方案（hosts中添加内容）。
* clone 官方源码：`git clone https://github.com/facebook/react.git`
* clone yck大神源码：`git clone https://github.com/KieSun/react-interpretation.git`

## React源码目录
进入react项目之后，再进入`packages`目录，这里存放的代码就是我们需要学习的内容，它的结构大致如下。

**react/packages目录结构**
```
├── create-subscription
├── eslint-plugin-react-hooks
├── jest-mock-scheduler
├── jest-react
├── legacy-events `事件系统`
├── react `react核心包`
├── react-art
├── react-cache
├── react-debug-tools
├── react-devtools
├── react-devtools-core
├── react-devtools-inline
├── react-dom `dom相关`
├── react-events
├── react-is
├── react-native-renderer
├── react-noop-renderer
├── react-reconciler `react-dom与react-native等共用`
├── react-refresh
├── react-stream
├── react-test-renderer
├── scheduler `调度：异步渲染等`
├── shared `共享`
└── use-subscription
```

我们可以看到react将不同的功能封装为不同的包，有些包可以为其他包所公用，一些重要的且功能独立的功能还可以发布为单独的npm包，我们可以通过研究其模块化拆分、组合等技巧，万一哪天我们自己也做了个很火的开源项目呢，代码得漂亮一点。仔细观察`16.9.0`目录结构跟`16.8.6`是有些不一样的，事件系统也由`events`变成了`legacy-events`，具体的变化我们后面仔细研究（推荐学习的时候顺便再撸一遍[官方文档（英文）](https://reactjs.org/docs/hello-world.html)）。本文写作时React英文文档`16.9.0`,中文文档`16.8.6`。

## JSX与React.createElement
对于学习React源码的同学必定多少了解和使用过JSX，这里就不对它进行描述了，有需要的可以去[React官网—JSX简介](https://react.docschina.org/docs/introducing-jsx.html)进行阅读了解。

官方说`JSX 可以生成 React “元素”`,那么它是如何生成React元素的？我们可以通过[babel在线编译（点击进入）](https://babeljs.io/repl/)进行了解

![babel在线编译界面](https://www.github.com/kingshuaishuai/static_resource/raw/master/assets/1565938718697.png)

**例1**

在左边可以输入JSX表达式，右边会帮助我们编译为React代码。

``` jsx
// 输入
<div id="hello" class="hello" key="hello">Hello React</div>
```
输入上述JSX代码，babel会帮助我们编译为下方代码

```javascript
// 输出
"use strict";

React.createElement("div", {
  id: "hello",
  class: "hello",
  key: "hello"
}, "Hello React");
```
通过编译后的代码我们可以知道，为什么在写React组件时，明明没有用到React却需要引入，否则就会报错，这是因为JSX的本质还是`React.createElement()`，它只是这个API的语法糖，不引入React就无法使用这个API。

仅仅是看编译后的代码，我们也能看出目前编译出的`React.createElement()`传入了三个参数：
1. 第一个是div元素的名称。
2. 第二个是元素的属性，其为一个对象，元素的所有属性都被编译到这个对象中了。
3. 第三个是这个元素中所包含的内容，当前是一个字符串。

想想如果是我们自己，我们会用怎样的数据结构来表示一个HTML元素，其实也不过就是元素名，属性和内容了，所以以上三个参数也是很好理解的。

**例2: 写一个稍微复杂点的JSX表达式试试**

``` jsx
// 输入
<ul id="hello" class="hello" key="hello">
	<li>Hello React</li>
	<li>Hello JSX</li>  
</ul>
```

```javascript
// 输出
"use strict";

React.createElement("ul", {
  id: "hello",
  class: "hello",
  key: "hello"
}, React.createElement("li", null, "Hello React"), React.createElement("li", null, "Hello JSX"));
```
从编译结果我们可以看到，`React.createElement()`传入的参数总数变了，变成了四个，前两个参数仍是元素名称和属性，后两个都是`React.createElement()`，这两个函数的参数又跟我们第一个例子一样了。

**例3：接下来写个React函数组件看看什么效果**

``` jsx
// 输入
function Wrapper({children}) {
	return <div class="wrapper">{children}</div>
}
<Wrapper>
  <ul id="hello" class="hello" key="hello">
	<li>Hello React</li>
	<li>Hello JSX</li>  
  </ul>
</Wrapper>
```

```javascript
// 输出
"use strict";

function Wrapper(_ref) {
  var children = _ref.children;
  return React.createElement("div", {
    class: "wrapper"
  }, children);
}

React.createElement(Wrapper, null, React.createElement("ul", {
  id: "hello",
  class: "hello",
  key: "hello"
}, React.createElement("li", null, "Hello React"), React.createElement("li", null, "Hello JSX")));
```
这里写了一个简单的`Wrapper`组件，在编译后的结果相信大家可以看得懂了，只不过细心的同学可以看到第一个参数不再是一个字符串，而是`Wrapper`变量，如果我们在使用`Wrapper`时候将其首字母改为小写`wrapper`,此时编译结果为

``` jsx
"use strict";

function Wrapper(_ref) {
  var children = _ref.children;
  return React.createElement("div", {
    class: "wrapper"
  }, children);
}

React.createElement("wrapper", null, React.createElement("ul", {
  id: "hello",
  class: "hello",
  key: "hello"
}, React.createElement("li", null, "Hello React"), React.createElement("li", null, "Hello JSX")));
```
最外层第一个参数变成了字符串，这里要说明的是为什么写React时要求我们将组件首字母大写，如果是小写，会编译为字符串，匹配原生的HTML标签，所以会出错，首字母大写才是组件。

## 源码初探——createElement

通过babel的在线运行平台，我们看清楚JSX转换到JavaScript的秘密。接下来可以试试查看源码：
打开`packages/react/src/React.js`(基于16.9.0), 20-26行引入了`ReactElement.js`中的内容

``` javascript
import {
  createElement,
  createFactory,
  cloneElement,
  isValidElement,
  jsx,
} from './ReactElement';
```
其中就包含我们刚刚所看到的`createElement`，我们可以打开`packages/react/src/createElement.js`，定位到`createElement`导出的地方，看看这个API如何实现。


代码上方有段注释`Create and return a new ReactElement of the given type.`表明这个API作用是根据所给出的`type`类型创建并返回了一个新的`ReactElement`类型的元素。并提供了[API说明文档](https://reactjs.org/docs/react-api.html#createelement)有兴趣的同学可以先查看一下该API的描述和使用方式再看实现。


**代码片段1：函数定义**
``` javascript
export function createElement(type, config, children) {
  let propName;

  // Reserved names are extracted
  const props = {};

  let key = null;
  let ref = null;
  let self = null;
  let source = null;
  ...
}
```
先来看函数和内部变量的定义，函数的三个参数在我们使用babel进行编译的时候就猜到了是什么意思：
* `type`：函数会根据type的不同区创建不同种类的`ReactElement`(react元素)，最简单的理解就是我们第三个例子，当使用首字母小写的时候第一个参数为字符串、首字母大写的时候第一个参数是一个变量，react会根据不同的类型给我们创建不同的react元素。
* `config`：这个参数用来描述元素的属性,根据编译结果我们也知道它是一个对象类型，元素的每个属性可以对应里面一个`key-value`对，例如`id`,`className`,`key`,`ref`等都在config中进行描述。
* `children`：代表元素的内容或者子元素，但是有奇怪的是我们之前看到的函数有可能会有第四、第五、第六个参数啊，这里怎么用一个children来表示？不妨想想，如果用一个children来表示从第三个元素开始后面的元素，代码中你会怎样做？我想大家都猜到了，可以用`arguments`参数来进行截取。如果是你来实现，你会有其他方法嘛？比如第三个参数使用`rest参数(...children)`，在函数中我们就可以不用截取。不过这里为什么没有使用这种方式还需要大家自己去探索和思考一下。

然后就是一些变量的定义了，可以看到的是尽管这是JavaScript代码，可以随时定义变量，作者仍将变量的定义提前，这也是值得我们学习和实践的。看源码的过程中希望大家尽量注意作者给出的注释。`Reserved names are extracted`表明这里会将保留名提取出来，因此我们可以猜到在props中只会存在一些正常的属性，特殊的内容会被过滤掉，后面必定也有相关逻辑。

**代码片段2：元素属性过滤**

``` javascript
  if (config != null) {
    if (hasValidRef(config)) {
      ref = config.ref;
    }
    if (hasValidKey(config)) {
      key = '' + config.key;
    }

    self = config.__self === undefined ? null : config.__self;
    source = config.__source === undefined ? null : config.__source;
    // Remaining properties are added to a new props object
    for (propName in config) {
      if (
        hasOwnProperty.call(config, propName) &&
        !RESERVED_PROPS.hasOwnProperty(propName)
      ) {
        props[propName] = config[propName];
      }
    }
  }
```

![流程图](https://www.github.com/kingshuaishuai/static_resource/raw/master/assets/1566005551482.png)

这里画了一张不标准的流程图（大致描述代码），如果有强迫症的同学可以动手画一画做的标准一点。从流程图我们可以比较清楚的看到这段代码的作用，首先从`config`中过滤出`ref`和`key`属性，这两个属性应该是保留属性中的内容，然后获取了`config`的`__self`和`__source`属性，之后对config的自有属性进行了遍历，过滤了保留属性中的内容，将其他属性存放在`props`对象中。
一句话将就是获取普通属性放到`props`中并提取了保留属性。上面提到的`__self`和`__source`长什么样子，我们可以看看16-21行的定义中看到。

``` javascript
const RESERVED_PROPS = {
  key: true,
  ref: true,
  __self: true,
  __source: true,
};
```

**代码片段3：提取children**

``` javascript
  // Children can be more than one argument, and those are transferred onto
  // the newly allocated props object.
  const childrenLength = arguments.length - 2;
  if (childrenLength === 1) {
    props.children = children;
  } else if (childrenLength > 1) {
    const childArray = Array(childrenLength);
    for (let i = 0; i < childrenLength; i++) {
      childArray[i] = arguments[i + 2];
    }
    if (__DEV__) {
      if (Object.freeze) {
        Object.freeze(childArray);
      }
    }
    props.children = childArray;
  }
```

![流程图](https://www.github.com/kingshuaishuai/static_resource/raw/master/assets/1566007585601.png)

从`代码片段3`可以看出首先通过arguments对象长度计算`children`的长度，若`children`只有一个元素则props对象的children属性就指向这个元素，如果children元素个数大于1，则创建childArray对象存储他们，再让props对象的children元素指向这个数组。
`注意事项：`
从上述源码中我们可以看出来`props.children`可能是一个数组，也可能不是，所以在使用`children`过程中，我们必须注意`children`是不是数组

> 其中有段代码是冻结`childArray`对象，对于`Object.freeze()`，平时使用并不多，如果小伙伴不知道它的作用，看看下面取自[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/freeze)的一段描述：
> `Object.freeze()` 方法可以冻结一个对象。一个被冻结的对象再也不能被修改；冻结了一个对象则不能向这个对象添加新的属性，不能删除已有属性，不能修改该对象已有属性的可枚举性、可配置性、可写性，以及不能修改已有属性的值。此外，冻结一个对象后该对象的原型也不能被修改。

**代码片段4：取默认属性**

``` javascript
  // Resolve default props
  if (type && type.defaultProps) {
    const defaultProps = type.defaultProps;
    for (propName in defaultProps) {
      if (props[propName] === undefined) {
        props[propName] = defaultProps[propName];
      }
    }
  }
```

![流程图](https://www.github.com/kingshuaishuai/static_resource/raw/master/assets/1566009656448.png)

这段代码比较容易看懂，如果`type`中存在默认属性，则遍历取出来，如果`props`对象中没有该属性则进行存入。而这个拥有`defaultProps`的`type`是什么，我们之前看到过，type可以为字符串:表示一个html标签；也可以为变量：表示一个组件，那一个组件中就可能存在一些属性了。

**代码片段5：结束**

``` javascript
  if (__DEV__) {
    if (key || ref) {
      const displayName =
        typeof type === 'function'
          ? type.displayName || type.name || 'Unknown'
          : type;
      if (key) {
        defineKeyPropWarningGetter(props, displayName);
      }
      if (ref) {
        defineRefPropWarningGetter(props, displayName);
      }
    }
  }
  return ReactElement(
    type,
    key,
    ref,
    self,
    source,
    ReactCurrentOwner.current,
    props,
  );
```
在这个函数中还剩下最后一段代码（本来不想说这个了，但是强迫症逼的还是说一下吧），在开发环境下（`__DEV__`），如果`key`跟`ref`任何一个属性存在，则会判断type的类型是否为`function`，如果是的话，则取出`type`的`displayName`或`name`，如果这两个属性都没有则取`Unknown`,如果不是`function`,就取`type`（此时指的是这个html标签名称），取出来的作用就是在`key`或者`ref`爆出警告时候，展示出来的名称就是这个`displayname`。最后根据一系列属性返回一个`ReactElement`;

既然说到了`ReactElement`，我们就先来看看这个API。

## ReactElement: 探索React元素

``` javascript
const ReactElement = function(type, key, ref, self, source, owner, props) {
  const element = {
    // This tag allows us to uniquely identify this as a React Element
    $$typeof: REACT_ELEMENT_TYPE,

    // Built-in properties that belong on the element
    type: type,
    key: key,
    ref: ref,
    props: props,

    // Record the component responsible for creating this element.
    _owner: owner,
  };

  if (__DEV__) {
    // The validation flag is currently mutative. We put it on
    // an external backing store so that we can freeze the whole object.
    // This can be replaced with a WeakMap once they are implemented in
    // commonly used development environments.
    element._store = {};

    // To make comparing ReactElements easier for testing purposes, we make
    // the validation flag non-enumerable (where possible, which should
    // include every environment we run tests in), so the test framework
    // ignores it.
    Object.defineProperty(element._store, 'validated', {
      configurable: false,
      enumerable: false,
      writable: true,
      value: false,
    });
    // self and source are DEV only properties.
    Object.defineProperty(element, '_self', {
      configurable: false,
      enumerable: false,
      writable: false,
      value: self,
    });
    // Two elements created in two different places should be considered
    // equal for testing purposes and therefore we hide it from enumeration.
    Object.defineProperty(element, '_source', {
      configurable: false,
      enumerable: false,
      writable: false,
      value: source,
    });
    if (Object.freeze) {
      Object.freeze(element.props);
      Object.freeze(element);
    }
  }

  return element;
};
```
这个函数的注释非常多，开头的一些没有贴上来，希望大家可以仔细看看这个函数相关注释。根据开头注释的描述，这个函数是个创建React元素的工厂方法，它不在支持类的模式，因此不能使用new操作符来新建元素，所以`instanceof`也无法对它的类型进行检查。那如果想要查看是否为React元素怎么办？这里提供了`Symbol.for('react.element')`的方式来对$`$$typeof`字段进行检查来确定是否为React元素。

忽略`if`语句，其实这段代码就是创建了一个`element`并将其返回，这个`element`元素中包含了一些属性，其中`type`、`key`、`ref`、`props`我们已经很了解了，那么`$$typeof:REACT_ELEMENT_TYPE`是个什么东西？根据上一段文字，我们可以知道它是用来判断一个元素是否为React元素的东西，我们可以顺着找到它的定义所在`packages/shared/ReactSymbols.js`:

``` javascript
export const REACT_ELEMENT_TYPE = hasSymbol
  ? Symbol.for('react.element')
  : 0xeac7;
```
对于这段代码的逻辑相信大家都看的懂，但是对于`Symbol.for`，大家平时用到应该不是很多，有些同学可能会对他感到陌生，下面引用[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Symbol/for)中一段描述来说明一下它的作用：
> Symbol.for(key) 方法会根据给定的键 key，来从运行时的 symbol 注册表中找到对应的 symbol，如果找到了，则返回它，否则，新建一个与该键关联的 symbol，并放入全局 symbol 注册表中。

还有一个`_owner`我们不太熟悉，但是根据注释我们就可以很清楚的知道它是负责记录创建此元素的组件的。通过它我们可以知道这个元素是哪个组件创建的。

> 我们一会说`元素`，一会说`组件`，可能有小伙伴会感觉很晕，什么是元素，什么又是组件啊？yck大神在他的文章中很清楚的告诉了我们：如果用JSX来写`<App/>`，那么`<App/>`就是ReactElement(React元素)，而`App`代表React Component(React组件)

我为什么把if语句也贴了出来，其实希望大家可以看看里面的注释（写的非常明确），不懂的话可以有道翻译，了解一下它做了什么。这里就不深入它了。

## 本章小结
1. 首先我们通过Babel在线运行工具查看了JSX转换的秘密，从而了解到为什么写JSX代码时候我们没用到React却要引入它，并了初步了解了`React.createElement()`的使用方式，以及它参数不同时会出现什么结果。
2. 然后我们深入了解了`React.createElement(type, config, children)`API的实现，总的来说，它干的事情并不是很多，过滤了一些保留的属性，将普通属性放到`props`中，通过对`children`的处理，将其放到`props.children`中，在使用时，我们一定要注意children是否为数组，最后返回一个React元素。
3. 最后我们看了`ReactElement()`的实现，它其实就是根据我们上面第二条中处理的一些参数来创建一个React元素，React元素的检验方式要通过他的`$$typeof`属性来检验，它还有个`_owner`属性表示创建它的组件。
4. 最后还留了个让大家自己查看的内容，希望大家可以配合源码、文档、有道词典来仔细研究源码。

**说在最后**
本篇内容其实也不算短了，本来想再写一点，但是这一篇写的时间有点长，也有点写不下去了，如果再长点，读者还没读完也就放弃了，还是放到后面文章里面在写吧。

再说说已经有大神在写React源码博客了，并且还有相关视频课程，我一个菜鸟写的有人看嘛?其实我就是这种想法，为什么不去看大神的要看一个菜鸟的。想了想，其实看源码、博客，每个人的理解和最终的表达方式都不可能是一样的，之前也好久没写文章了，因为总感觉我要写的别人都已经写过了，而且也写的很好，想着我再牛逼一点再写点高质量的文章。

不过现在感觉自己错了，大神的想法毕竟是大神的，大神的表达也是大神的，不一定每个人看理解的程度都能达到大神们的境界，每个人看文章也好，源码也罢，理解也可能是不同的，如果一个菜鸟能将自己的想法讲清楚，其实很大程度上也可以帮助到跟自己同一个level或者更低level的人更清楚地学习这些知识。而且阅读、理解是输入，写作是输出，在输出中会想清楚更多的东西。

在这里也非常鼓励大家将自己的学习分享出去，我们一起勇敢一点、大不了写的太烂结果就是没人看嘛，顶多再嘲讽两句，但自己学到的东西会不断推着我们向前。

本系列博客GitHub地址：[https://github.com/kingshuaishuai/blog](https://github.com/kingshuaishuai/blog)
由于时间原因，更新不稳定，但一定会持续，希望有志同道合的小伙伴们相互督促，一起学习，明天的我们一定会感谢今天努力的自己。