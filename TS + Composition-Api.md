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
    ![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0dffd735ae064e939e770ddea9bdde66~tplv-k3u1fbpfcp-zoom-1.image)
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

babel.config.js稍作修改
```javascript
module.exports = {
  presets: ["vca-jsx", "@vue/cli-plugin-babel/preset"]
};

```

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

## `defineComponent`初体验

首先改写HelloWorld组件

```typescript
<template>
  <div class="hello">
    Hello world
  </div>
</template>

<script lang="ts">
import { defineComponent, getCurrentInstance } from "@vue/composition-api";

export default defineComponent({
  name: "HelloWorld",
  props: {
    msg: String
  },
  setup(props, ctx) {
    console.log(getCurrentInstance());
    console.log(ctx);
  }
});
</script>
```
通过`defineComponent`进行组件的定义，setup函数有两个常用参数，第一个为props,第二个为setupContext, 这两个值跟vue3是一样的，可以通过`getCurrentInstance`获取当前组件实例，这个函数返回值为当前组件实例，打印出来后跟vue2的`this`内容是一样的，之前该有的参数都还在，只不过`setup`中没有`this`,只有ctx,这也够用了。有兴趣可以看看控制台都打印出了什么东西。

## TSX体验
`src/compnents/TestComp.tsx`以tsx方式创建TestComp组件,注意属性`comp`会接收一个组件，我们可以在props中规定类型为Object,但这并不够，我们需要确定comp详细的类型，那就可以在setup中重新规定一下props的类型。

```typescript
import { defineComponent } from "@vue/composition-api";
import { VueConstructor } from "vue/types/umd";

type TestCompProps = {
  comp: VueConstructor<Vue>
}
export default defineComponent({
  name: "TestComp",
  props: {
    comp: {
      type: Object
    }
  },
  setup(props: TestCompProps) {
    const { comp: Comp } = props;
    return () => <Comp />;
  }
});
```

或者直接规定在defineComponent的泛型参数中

```typescript
import { defineComponent } from "@vue/composition-api";
import { VueConstructor } from "vue/types/umd";

type TestCompProps = {
  comp: VueConstructor<Vue>
}
export default defineComponent<TestCompProps>({
  name: "TestComp",
  props: {
    comp: {
      type: Object
    }
  },
  setup(props) {
    const { comp: Comp } = props;
    return () => <Comp />;
  }
});
```

更多玩法请直接command + 点击或ctrl + 鼠标点击进入`defineComponent`声明文件进行探索。

注意我们每次定义完一个组件后鼠标指上去看看是什么类型，经过观察其实是`VueConstructor<Vue>`类型，这样在return 时候使用tsx的用法才不会报错。

**通过属性传入组件**
`src/compnents/AA.vue`创建AA.vue组件，写个普通的vue组件。

```typescript
<template>
  <div class="hello">
    AA Component
  </div>
</template>

<script lang="ts">
import { defineComponent } from "@vue/composition-api";

const HelloWorld = defineComponent({
  name: "AA"
});
export default HelloWorld;
</script>

```

在HelloWorld中引入AA组件与TestComp组件，然后将AA传递给TestComp

```typescript
<template>
  <div class="hello">
    Hello world
    <test-comp :comp="AA"></test-comp>
  </div>
</template>

<script lang="ts">
import { defineComponent, getCurrentInstance } from "@vue/composition-api";
import TestComp from "./TestComp";
import AA from "./AA.vue";

export default defineComponent({
  name: "HelloWorld",
  props: {
    msg: String
  },
  components: {
    TestComp,
    AA
  },
  setup(props, ctx) {
    console.log(getCurrentInstance());
    console.log(ctx);
    return {
      AA
    };
  }
});
</script>
```
此时可以看到浏览器可以输出AA Component字样，说明成功。

**tsx的另外一种写法**
再创建`MM.tsx`

```typescript
const MM = () => {
  return () => <div>this is MM</div>
}

MM.name = 'MM';

export default MM;
```
然后在HelloWord组件中引入它，同样的方法return出去（直接放在AA下面），然后将传递进TestComp组件的属性由AA替换为MM。保存，仍然OK。只不过此时代码不会报错但是`Vetur`插件会给我们报个红色波浪线。所以我还是推荐使用TestComp里的这种方式进行TSX组件定义。

其实这就是官方文档所说的setup返回一个函数的时候，这个函数会被当做render函数来使用，所以它就是vue2中的函数式组件了。

## 利用函数组件二次封装一个超级方便好用的表格组件

**重要：渲染自定义table单元格组件的容器 TableCellRender.tsx**

表格会传进来一个comp组件作为自定义的单元格，事先可能不知道啊这里要渲染什么，还会传进来scope数据

```typescript
import { defineComponent } from "@vue/composition-api";
import { VueConstructor } from 'vue/types/umd';

type TableCellRenderProps = {
  scope: any;
  comp: VueConstructor<Vue>
}

export default defineComponent<TableCellRenderProps>({
  name: 'TableCellRender',
  props: {
    scope: {
      type: Object,
      required: true
    },
    comp: {
      type: Object,
      required: true
    }
  },
  setup(props) {
    const { comp: Comp } = props;
    console.log('props.scope', )
    return () => <Comp row={props.scope.row} />
  }
})

```

**TableBase.vue组件**
通用组件，定义了四种单元格，一种为link类型的，一种为多选框，一种为自定义传进来的动态组件，最后一种为默认组件，外加一个翻页器，当然翻页器可以被隐藏。

```typescript
<template>
  <div class="table-base">
    <div class="table-container">
      <el-table
        :size="size"
        v-loading="loading"
        :data="data"
        tooltip-effect="dark"
        style="width: 100%"
        @selection-change="handleSelectionChange"
      >
        <el-table-column
          v-if="multiple"
          type="selection"
          width="55"
          :selectable="checkSelectable"
        ></el-table-column>
        <template v-for="(column, index) in tableColumns">
          <el-table-column
            v-if="column.comp"
            :key="index"
            :prop="column.key"
            :label="column.label"
            :width="column.width ? column.width : ''"
            :show-overflow-tooltip="!column.multipleline"
          >
            <template slot-scope="scope">
              <table-cell-render
                :scope="scope"
                :comp="column.comp"
              ></table-cell-render>
            </template>
          </el-table-column>
          <el-table-column
            :key="index"
            v-else-if="column.active"
            :prop="column.key"
            :label="column.label"
            :width="column.width ? column.width : ''"
            :show-overflow-tooltip="!column.multipleline"
          >
            <template slot-scope="scope">
              <span
                class="active-link"
                @click="() => handleClickActiveLink(scope.row)"
                >{{ scope.row[column.key] }}</span
              >
            </template>
          </el-table-column>
          <el-table-column
            v-else
            :key="index"
            :prop="column.key"
            :label="column.label"
            :width="column.width ? column.width : ''"
            :show-overflow-tooltip="!column.multipleline"
          ></el-table-column>
        </template>
      </el-table>
    </div>
    <div class="table-pagination" v-if="!noPagination">
      <el-pagination
        class="pagination"
        background
        :layout="layout"
        :page-size="pageSize"
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="currentPage"
        :total="total"
      ></el-pagination>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent } from "@vue/composition-api";
import TableCellRender from "./TableCellRender";
export default defineComponent({
  name: "MTableBase",
  components: {
    TableCellRender
  },
  props: {
    layout: {
      type: String,
      default: "total, prev, pager, next, jumper"
    },
    size: {
      type: String,
      default: "small"
    },
    loading: {
      type: Boolean,
      default: false
    },
    multiple: {
      type: Boolean,
      default: true
    },
    tableColumns: {
      type: Array,
      default: () => []
    },
    data: {
      type: Array,
      default: () => []
    },
    pageSize: {
      type: Number,
      default: 10
    },
    pageSizes: {
      type: Array,
      default: () => []
    },
    currentPage: {
      type: Number,
      default: 0
    },
    total: {
      type: Number,
      default: 0
    },
    noPagination: {
      type: Boolean,
      default: false
    }
  },
  setup(props, ctx) {
    const { emit } = ctx;
    const handleSelectionChange = (val: any) => emit('selection-change', val);
    const handleSizeChange = (val: number) => emit('current-change', val);
    const handleCurrentChange = (val: number) => emit('current-change', val);
    const handleClickActiveLink = ($event: MouseEvent, row: any) => emit('get-row-info', $event, row);
    const checkSelectable = (row: any) => row.name !== 'None';

    return {
      handleSelectionChange,
      handleSizeChange,
      handleCurrentChange,
      handleClickActiveLink,
      checkSelectable
    }
  }
});
</script>

<style lang="scss" scoped>
.table-pagination {
  padding-top: 20px;
  .pagination {
    text-align: center;
  }
}
.table-container /deep/ {
  .el-table {
    font-size: 14px;
  }
}
</style>

```

**表格组件的使用**

将home页面改造为ts，并使用`defineComponent`定义组件。以后表格组件再也不用动了，每次只需要给特定的列定义自己的渲染组件就可以进行渲染了。

```typescript
<template>
  <div class="home">
    <HelloWorld msg="Welcome to Your Vue.js App" />
    <table-base :tableColumns="column" :data="data"></table-base>
  </div>
</template>

<script lang="tsx">
import { defineComponent } from "@vue/composition-api";
import HelloWorld from "@/components/HelloWorld.vue";
import TableBase from "@/components/TableBase.vue";

const helloCell = defineComponent({
  name: "HelloCell",
  props: {
    row: {
      type: Object
    }
  },
  setup(props: { row: { hello: string } }) {
    console.log("cell inner", props);
    const hello = props.row.hello;

    return () => <el-button type="primary" size="mini">{hello}</el-button>;
  }
});

export default defineComponent({
  name: "Home",
  components: {
    HelloWorld,
    TableBase
  },
  setup() {
    const column = [
      { label: "Hello", key: "hello", comp: helloCell },
      { label: "World", key: "world" }
    ];
    const data = [
      { hello: "hi", world: "wd" },
      { hello: "hello", world: "world" }
    ]

    return {
      column,
      data
    }
  }
});
</script>
```
效果：
![](https://github.com/kingshuaishuai/static_resource/blob/master/assets/1596741030957.png?raw=true)

> 问题：
> 目前看似可以了，但是眼尖的小伙伴肯定会发现一些猫腻，在`TableCellRender`中定义的Comp属性规定类型为`VueConstructor<Vue>`，此时它没有定义props，所以row下面会有红色波浪线，这里暂时没理清怎么做，不过不会影响项目编译运行。如果有弄明白的可以下面留言解答一下。

## hooks助力解耦公用逻辑与复杂逻辑拆分

vue3 / composition-api拥抱函数式编程，我们使用新的技术也需要做开发方式的转换，如果vue3到时候还是跟vue2一模一样的写法和使用，那么还不如继续使用2呢。

**hooks的使用场景**

**1. 拆分公用逻辑**

用一个真实场景来吧，这几天我们的系统有个小问题，dialog弹出框的每个form表单都需要点开后自动聚焦在第一个input上，然而Element虽然提供了`autofocus`的属性，但它并不会自动明聚焦。这就需要手动维护`ref`，在mounted后，通过在`nextTick`中手动调用组件的`focus()`方法，只不过要改的组件很多，一个一个加太费力了。所以只能使用`mixin`,然后在每个dialog的首个input添加`ref`为`autofocus`的属性。

```javascript
export default {
  name: 'AutoFocusMixin',
  mounted() {
    this.$nextTick(() => {
      this.$refs.autofocus.focus();
    });
  }
};
```

这样做的好处很明显，共享了代码逻辑，但是后人维护时候可能会很蒙蔽，看到ref="autofocus"但是直接在文件中搜索却不能找到哪里用了它，如果没注意到mixin，那么删除了这个属性可能还会以为优化了代码，最后只会导致问题重现。

但是当vue有了hook，一切就不一样了，我们可以将这段逻辑提取出来

```typescript
// useAutofocus.ts
import { ref, Ref, onMounted } from '@vue/composition-api';
import { Input } from 'element-ui';

export function useAutofocus() {
  const focusEl:Ref<null | HTMLInputElement | Input> = ref(null);

  onMounted(() => {
    setTimeout(() => {
      if (focusEl.value) {
        focusEl.value.focus();
      }
    }, 0)
  })

  return focusEl;
}

```

在需要使用的组件中引入

about.vue
```typescript
<template>
  <div class="about">
    <el-input ref="focusEl" placeholder="请输入内容" v-model="inputValue"/>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref } from "@vue/composition-api";
import { useAutofocus } from "@/hooks/useAutofocus";
export default defineComponent({
  name: "About",
  setup() {
    const inputValue = ref("");
    const focusEl = useAutofocus();

    return {
      inputValue,
      focusEl
    };
  }
});
</script>

```

![enter description here](https://www.github.com/kingshuaishuai/static_resource/raw/master/assets/1596772908309.png)

autofocus生效，完美。这样比mixin的好处就很明显了，最起码我们可以找到变量在哪里定义的，怎样使用的，避免维护上的模糊与困难。

此外还有一点，就是mixin有时候会写很多的逻辑，但是hooks你可以尽管往细了拆分，你最终需要谁就引入谁进去。

**2. 拆分复杂逻辑**

如果你的项目非常复杂，在一个页面中可能写上千行的代码，那么安小功能可以将你每个功能代码拆分到hooks中，依赖的数据通过参数进行传递，当然，hooks也可以返回多种多样的数据类型，比如函数，可以用个hook来写你的点击或者其他操作的业务逻辑，最终返回一个函数，点击时调用它。

有些极端的小伙伴甚至能将所有的业务逻辑全部拆分到hooks中，组件中只会留下一堆创建变量，导出变量和引用变量的信息。

拆分逻辑后，有可能在别的地方也会使用这些hooks，就算用不到，这也会给维护带来更多的便利性。毕竟一些函数一会写在`mounted`中一会又要在`updated`中写，乱七八糟一种逻辑分散在各处，维护起来成本也是挺大的。

## vue-composition-helpers的使用