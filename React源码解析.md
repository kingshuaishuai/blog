[TOC]
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

## 源码初探
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

代码上方有段注释`Create and return a new ReactElement of the given type.`表明这个API作用是创建并返回了一个新的`ReactElement`类型的元素。并提供了[API说明文档](https://reactjs.org/docs/react-api.html#createelement)有兴趣的同学可以先查看一下该API的描述再看实现。
