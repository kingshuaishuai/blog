---
title: TS + Composition-Api 
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---

## 前言
好久没输出了，今天来输出一把，缓解一下一个人的孤独。

Vue3虽然还没正式发布，但公布到现在也是蛮久了，虽然现在已经可以开始尝鲜，但由于周边生态还不完善，并且proxy无法被polyfill,导致它也不能支持IE11,如果是一些2C的产品使用了vue进行开发，就算3出了，可能也会由于考虑IE用户暂时无法升级。还有很多利用ElementUI等Vue2.x框架的产品，短时间内也生态不完善也不太容易转移到vue3。

不过值得高兴的是，vue3的核心功能composition-api是同时支持vue2与vue3两个主版本的。我们已经可以在一些小项目中尝试使用composition-api来做开发了，体验与vue3基本一致，只不过不能用teleport,suspense等新的功能，但并不影响coding的愉悦之感。在尤大大刚直播vue3之后，跟很多小伙伴一样，迫不及待地进行了把玩，然而发现由于破坏性的改动导致例如elementUI等框架无法与vue3进行配合，虽然网上有说法利用cdn引入vue2.x兼容ElementUI，自己的组件可以使用vue3来写，当然这样玩玩可以，但总让人有点不舒服的感觉。其实倒也不必可以追求3，因为我们完全可以使用vue2.x + composition-api的方案来进行开发，并且兼容ElementUI等Vue2.x的UI框架。

由于本人所在团队只有我一个前端，技术的选择也是无比自由，最近也是用vue2.6 + composition-api + ts重构了一个项目，做了一个新的小项目，今天又尝试了一把使用这个方案做组件库（抽离出的公共功能做个小组件库），遇到了一些问题，但幸运地给解决掉了，又搞出了之前这个方案中遇到的JSX相关问题，所以抑制不住激动的心情，晚上还是出来分享一下最近的使用体验吧。（其实是一个人太孤独了，想找小姐姐聊天又找不到，孤独到难受，来写写文章舒缓一下心情）。

## 工欲善其事必先利其器

这里介绍一下vue-cli项目的创建，如果非常熟悉请跳过直接往后看。

**创建项目**

话不多说，接下来我们就一起用vue-cli创建一个ts项目，开始前请保证你的vue-cli是最新版本。

1. `vue create athena`创建一个项目（起名雅典娜），雅典娜女神比较著名，以此祝我早日找到自己的女神吧。
2. 接下来的选择比较重要，如果一直只是在公司大佬们创建的项目中新增功能，自己vue-cli用的比较少那还是要注意下的。选择最后一项`Manually select features`回车，我们需要自定义配置，不使用默认配置。
3. 这里推荐一个我比较常用的一个项目依赖内容的选项组合吧，这几个选项估计大家也都明白是干啥的，只不过我写测试比较少，E2E更是没写过，所以一般不选，如果有需要也可以自己看情况处理。
    ![enter description here](https://www.github.com/kingshuaishuai/static_resource/raw/master/assets/1596726353613.png)
4. 选中之后回车，接下来会被问道`Use class-style component syntax? (Y/n)`是否使用`class-style`语法，当然选择`N`啊，我们会完全使用composition-api，不会借助class来做，并且我个人不是很喜欢使用装饰器跟类这一套方案，如果有喜欢的，应该有好些资料介绍的，这里不选它。
5. 之后就会问你是否使用TS`Use Babel alongside TypeScript (required for modern mode, auto-detected polyfills, transpiling JSX)?`，默认是就可以。
6. 然后问你使用hash路由还是history路由，`Use history mode for router?`,看自己项目需求吧，不想额外配置nginx可以使用hash路由，这里我就默认了。
7. 然后就是询问使用哪个css预处理器了，`Pick a CSS pre-processor (PostCSS, Autoprefixer and CSS Modules are supported by default):`, 这里我选择第二个node-sass，因为我用sass比较多，有感情，dart-sass尝试过，深度选择器支持得不友好,所以不用。
8. 现在被问到的是linter和formatter的选择，这里推荐倒数第二个`ESLint + Prettier`，当然如果你有特殊需要选自己喜欢的就行了。
9. 然后会问啥时候去lint格式化你的代码，` Pick additional lint features:`， 两个选项保存时跟commit都选上就可以了。
10. 测试框架jest,特殊需求请自己选择
11. babel,eslint等配置放在哪里？ ` Where do you prefer placing config for Babel, ESLint, etc.?`，当然是单独的文件夹呀，都放在package.json里咋维护啊。

以上选择就是我通常的配置，各位可以根据需求自行选择。

**安装composition-api**
`yarn add @vue/composition-api`

在`src/main.ts`中进行引入使用.
```typescript
...
import CompositionAPI from '@vue/composition-api';

Vue.use(CompositionAPI);
...
```

**安装官方推荐jsx工具**
官方推荐了个jsx的工具，这个也需要安装`yarn add babel-preset-vca-jsx -D`

安装到dev依赖就行，打包后线上跑是不需要它的。当然不安装它也行，只不过涉及将将组件作为props传递给另外一个组件就不知道你该怎么做了。这个在后面二次封装一个超级方便的通用table组件非常重要。

**vuex插件安利**
在使用composition-api过程中，发现了`vuex-composition-helpers`神器，直接使用useXXXX函数，可以将vuex的state,actions, mutations,getters映射为响应式对象，用来代替常用的`mapState,mapActions,mapMutations,mapGetters`，当然，vuex中拆分的`modules`子store也有相应的`useNamespacedXXX`来替代。笔者最开始的时候还傻乎乎自己写了个`useStats, useActions, useStore`，然后坐地铁回家时突然就看到了这个工具，简直是欣喜若狂啊，有兴趣的小伙伴还可以去看看源码，实现的很简洁清晰明了。

然后要做的当然是安装一下了
`yarn add vuex-composition-helpers`

**安装ElementUI**
这就不多说了，官网打开，按教程安装并配置好
安装： `yarn add element-ui`
主题推荐创建一个scss文件：assets/style/_element-variables.scss，还可以很容易去覆盖一些主题色什么的。然后创建一个index.scss将这个文件引入，最后在main.ts中将scss文件引入就有了可配置的主题。
```scss
/* 改变 icon 字体路径变量，必需 */
$--font-path: '~element-ui/lib/theme-chalk/fonts';

@import "~element-ui/packages/theme-chalk/src/index";
```

```typescript
// main.ts
...
import ElementUI from 'element-ui';
import './assets/styles/index.scss'; // index.scss里包含element主题，也可以放一些reset的样式，公共样式或者其他

Vue.use(ElementUI, {
  size: 'small'
});
...
```

基本工具安装好了，接下来就可以愉快的coding了。

