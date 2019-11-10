# 前端SKU算法的实现

本文将提供一种前端SKU算法的实现思路，由于笔者第一次尝试实现SKU，因此这可能并不是最佳的实现方式，但可以为没有思路的小伙伴提供一种解决方案。

对于SPU与SKU概念还不了解的小伙伴，请先移步[认识SKU与SPU](https://www.github.com/kingshuaishuai/blog/blob/master/%E8%AE%A4%E8%AF%86SKU%E4%B8%8ESPU.md)了解一下大致概念，本文便不再详述。

## 实现效果预览
![](https://www.github.com/kingshuaishuai/static_resource/raw/master/assets/第六次作业.gif)

## 实现思路
对于前端来说，如果没真的做过SKU，可能并不会了解到它的复杂之处，仔细观察，小小的规格选择，却包含着很多实现细节难点，这里提供一种实现思路供大家参考。

### 1. 获取后端数据
这里以[7七月老师提供的商品数据](https://course.7yue.pro/lin/sleeve/3%20API%EF%BC%9ABanner.html#spu-%E5%95%86%E5%93%81)为例，由于数据过长，这里只贴出`sku_list`中的一条数据，这条数据格式也是整个实现过程中最重要的数据格式。

``` javascript
 {
    ...
    sku_list: [
        {
          "id":2,
          "price":77.76,
          "discount_price":null,
          "online":true,
          "img":"",
          "title":"金属灰·七龙珠",
          "spu_id":2,
          "category_id":17,
          "root_category_id":3,
          "specs":[
              {
                  "key_id":1,
                  "key":"颜色",
                  "value_id":45,
                  "value":"金属灰"
              },
              {
                  "key_id":3,
                  "key":"图案",
                  "value_id":9,
                  "value":"七龙珠"
              },
              {
                  "key_id":4,
                  "key":"尺码",
                  "value_id":14,
                  "value":"小号 S"
              }
          ],
          "code":"2$1-45#3-9#4-14",
          "stock":5
        },
        ...
    ],
    ...
 }
```

**新建一个类Spu模拟从后端获取数据**

``` javascript
class Spu {
  data = []

  constructor() {
    this.data = this.getData();
  }

  getData() {
    const data = sku_list;  //这里的sku_list就是七月老师提供的数据中对应字段的数据，我们目前只需要这些就可以了
    return data
  }
}
```

### 2.提取规格数据
后端给我们的数据中只有**单品---不同规格组合好的商品**列表,我们需要从单品列表中获取到所有的规格。这里我们可以从数据中看到每个单品的`specs`属性就是这个单品所包含的不同规格，我们需要将其提取出来。

``` javascript
currentSkuList = []

_getCurrentSkuList() {
  this.currentSkuList = this.data.map(item => item.specs);
}
```

**提取结果如下**

![](https://www.github.com/kingshuaishuai/static_resource/raw/master/assets/1573389088106.png)

### 3.矩阵转置
我们提取到的数据格式为

```
颜色     图案     尺码
金属灰   七龙珠   小号 S
青芒色   灌篮高手 中号 M
青芒色   圣斗士   大号 L
橘黄色   七龙珠   小号 S
```

但根据效果图，我们需要如下这种格式的数据，因此我们就需要将上面这个`矩阵`转置，它可能不是一个标准的矩阵，因为每种规格的数目不一定相同，这里相同可以更好的理解。

![enter description here](https://www.github.com/kingshuaishuai/static_resource/raw/master/assets/1573389367305.png)

``` javascript
_transMatrix() {
    this._getCurrentSkuList();

    let transResult = {};

    this.currentSkuList.forEach(specs => {
      specs.forEach(item => {
        if(!transResult[item['key_id']]) {
          transResult[item['key_id']] = {
            key_id: item['key_id'],
            key: item['key'],
            value_list: {
              [item['value_id']]: {
                value_id: item['value_id'],
                value: item['value'],
                selected: false,
                disabled: false
              }
            }
          }
        } else if (!transResult[item['key_id']].value_list[item['value_id']]) {
          transResult[item['key_id']].value_list[item['value_id']] = {
            value_id: item['value_id'],
            value: item['value'],
            selected: false,
            disabled: false
          }
        }
      })
    })
    return transResult;
  }
```

**转换结果**

![](https://www.github.com/kingshuaishuai/static_resource/raw/master/assets/1573389576879.png)

这里利用了JavaScript对象的灵活性，设置了一种数据结构

``` javascript
{
    key_id: { //规格id
        key: string, // 规格名称 
        key_id: number, // 规格id
        [value_list: object]: {  // 规格值列表 object
            value_id: { // 规格值id
                value_id: number, // 规格值id
                value: string, // 规格值
                selected: boolean, // 是否选中
                disabled: boolean, // 是否不可选
            }
        }
    },
    key_id: {...},
    ...
    
}
```

通过对`currentSkuList`当前单品列表的遍历和筛选，将其提取为上述数据格式的数据，这里使用对象而不是数组的方式是为了更方便的进行提取

**将提取结果变为数组格式**

``` javascript
  getAllSpecsList() {
    const transResult = this._transMatrix();
    this.allSpecsList = Object.keys(transResult).map(key => {
      let obj = JSON.parse(JSON.stringify(transResult[key]));
      obj.value_list = Object.keys(obj.value_list).map(vk => obj.value_list[vk]);
      return obj;
    })
    return this.allSpecsList;
  }
```
将转置的矩阵变为数组，方便我们在页面中使用`wx:for`循环，如果使用的`react`或`vue`来做循环，使用数组也是很方便的，如果直接使用上面的对象格式，还要再做一次转换数组，所以为了方便直接使用，我们再做一遍转换。

上面使用了`JSON.parse(JSON.stringify(obj))`的方式对转置后的矩阵对象做了一个深拷贝，通过`Object.keys()`与`Array.map`API很方便就可以将之前的对象转换为数组

**转换结果**

![enter description here](https://www.github.com/kingshuaishuai/static_resource/raw/master/assets/1573390520765.png)

`Tips：`
注意这里不是二维数组，而是之前我们设计的数据结构的对象数组，使用这种数据结构可以是我们后续轻易获取我们想要的数据。

### 4.最重要的一步：遍历
这一步的标题实在不知道该叫什么，但这是我们整个SKU算法实现的最重要的一步。

**选择逻辑的设计**
对于选择一个规格，其他规格中只有对应可选的部分能选择，其余部分不可选，直接的视觉效果就是置灰。但是如何进行联动，我们选择一个规格后，后面可能还有很多中规格，我们案例中选择一种之后还有两种，但事实上可能并不止如此，剩余的可能有七八中规格，每种规格甚至可能会有10多种值，每次选取的可能是这些规格中的任意一个，如果暴力循环所有的可能性，效果可能会很差。

这里提供一种思路，就是根据后端给的SKU列表来确定一种规格对应其他规格中可选的部分，描述不是很清楚，下面举个例子来说明：
知道了颜色规格为`青芒色`，那么其他规格中图案只有两种可以选：`灌篮高手`和`圣斗士`，尺码的规格只有`中号 M`和`大号 L`可选。

那么我们就可以确定这样一张表：
![](https://www.github.com/kingshuaishuai/static_resource/raw/master/assets/1573391861054.png)

那么是不是可以这样来想，我们每次点击选中一种规格之后，就去遍历其他规格，在遍历其他规格的时候，如果这种规格中的规格值不在我们的列表中，那么就给他置灰不可选（disabled），这样我们需要关心的事情就很简单了，再来捋一捋

我们每次点击一个规格值之后，我们不用关心这种规格了，比如选了青芒色，就不用关心其他颜色了，只用关心其他种类的规格，其他种类规格里面也不需要全部都关心，我们只需要关心这种规格中有哪个可以跟我当前点的这个匹配。

举例：
如上表中，确定了颜色是灰色，就去遍历其他种类的规格（`图案`和`尺码`），遍历图案的时候，只用知道`七龙珠`跟我匹配，其余不匹配的全部置灰，遍历尺码的时候，只要知道`小号 S`跟我匹配，其他一律置灰。

再选图案，就只能选七龙珠，选了七龙珠之后，再去遍历其他种类的规格，颜色中只有`青芒色`跟`金属灰`可以跟我匹配，其他全部置灰。

再选尺码，只能选S了，根据上述规则，再去置灰其他两种规格中不匹配的。

这样剩下的就是可选的了。

将上述思路转换为代码：

``` javascript
  getSelectable() {
    if(this.allSpecsList.length === 0) {
      this.allSpecsList()
    }

    if(this.currentSkuList.length === 0) {
      this.currentSkuList = this.data.map(item => item.specs);
    }
    
    const rowLength = this.allSpecsList.length;

    for(let row = 0; row < rowLength; row++) {
      let { key_id, key } = this.allSpecsList[row];
      let columnList = this.allSpecsList[row].value_list;
      this.selectable[key_id] = {
        key_id,
        key,
        selectableList: {}
      }
      for (let column = 0; column < columnList.length; column++) {
        let { value_id, value } = columnList[column];
        this.selectable[key_id].selectableList[value_id] = {
          value_id,
          value,
          matchItems: null
        }
      }

      this.currentSkuList.forEach(specificSpecs => {
        let matchItems = {};
        let currentVlaueId = '';
        specificSpecs.forEach(specsItem => {
          if(specsItem.key_id !== key_id) {
            matchItems[specsItem.key_id] = [specsItem]
          } else {
            currentVlaueId = specsItem.value_id;
          }
        })

        if (!this.selectable[key_id].selectableList[currentVlaueId].matchItems) {
          this.selectable[key_id].selectableList[currentVlaueId].matchItems = matchItems;
        } else {
          Object.keys(this.selectable[key_id].selectableList[currentVlaueId].matchItems).forEach(k => {
            this.selectable[key_id].selectableList[currentVlaueId].matchItems[k].push(...matchItems[k])
          })
        }

      })
    }

    return this.selectable;
  }
```

**转换结果**
![](https://www.github.com/kingshuaishuai/static_resource/raw/master/assets/1573392661785.png)

这里又是借助JavaScript对象的灵活性设计了一种数据结构来表达我们上述的思路

``` javascript
{
    key_id: { // key_id 确定当前选了哪种类型的规格
        key: string, // 规格名
        key_id: number, // 规格id
        selectableList: { // 选择列表，用来描述我们上面画的表
            value_id: { // 当前选择了哪个规格，上面key_id确定了规格种类，这里value_id确定了这种规格中的具体规格,
                matchItems: {  // 当前规格对应其他规格的匹配项
                    key_id: [ // 当前规格匹配的种类规格的种类id，
                        {
                            key_id: numebr, // 种类id，这里为了方便数组中的直接取了后端返回的数据
                            key: string, // 规格种类名称
                            value_id: number, // 匹配项中的规格id
                            value: string, //匹配项中的规格名称
                        },
                        ...
                    
                    ]
                
                }
                
            }
            
        }
    },
    key_id: {...},
    ...
}
```

设计好数据结构之后，想办法通过后端数据给出的单品列表，确定每一个规格项可以匹配的其他规格，最终将数据转换为我们设计出来的数据结构。

### 5.书写简单的业务逻辑
上一步可谓是整个解决方案中最核心也是最难的地方了，只要获取了我们想要的数据结构，接下来就可以通过简单的业务逻辑来实现最后的效果。

**页面的data配置**

``` javascript
data: {
    sku: null,
    allSpecsList: [],
    selectable: null,
    selected: [],
    selectedItem: null,
    selectTips: '请选择：',
  }
```

**初始化数据**

``` javascript
initData() {
    const sku = new Sku();
    const allSpecsList = sku.getAllSpecsList();
    const selectable = sku.getSelectable();
    this.setData({
      allSpecsList,
      selectable,
      sku
    });
  },
```
初始化数据时候，我们创建Sku实例对象，将其保存在data中，再拿到`allSpecsList`（转置后的矩阵数组），来循环实现页面的规格展示。
拿到`selectable`方便后续处理点击某个规格时候对其他规格置灰的处理。

**实现页面展示**

``` javascript
<wxs src="../../wxs/specs.wxs" module="s"></wxs>
<view class="container">
  <view class="spu-info">
    <image class="selected-img" src="{{selectedItem && selectedItem.img ? selectedItem.img : 'http://i1.sleeve.7yue.pro/assets/5605cd6c-f869-46db-afe6-755b61a0122a.png'}}"></image>
    <view class="spu-container">
      <text class="spu-title">双色可选</text>
      <view class="spu-content">
        <view class="price">$1000</view>
        <view class="tips">
          {{selectTips}}
        </view>
      </view>
    </view>
  </view>
  <view class="select-options">
    <block wx:for="{{allSpecsList}}"
           wx:for-item="specs" 
           wx:for-index="x" 
           wx:key="specs.key_id"
    >
      <view class="specs-item">
        <text class="specs-title">{{specs.key}}</text>
        <view class="specs-value-list">
          <block wx:for="{{specs.value_list}}" 
                 wx:for-item="value" 
                 wx:for-index="y" 
                 wx:key="{{value.value_id}}"
          >
            <view class="value-container {{s.getButtonExtraClass(value.selected, value.disabled)}}" 
                  data-key_id="{{specs.key_id}}" 
                  data-value_id="{{value.value_id}}" 
                  data-select="{{s.getButtonStatus(value.selected, value.disabled)}}"
                  data-x="{{x}}"
                  data-y="{{y}}"
                  bindtap="handleClickSpecs"
            >
              <text class="value">{{value.value}}</text>
            </view>
          </block>
        </view>
      </view>
    </block>
  </view>
</view>
```

页面的实现主要就是我们定义的数据结构的循环遍历等操作，将数据展示过来。其实页面没什么好讲的，这里要说的主要是几个自定义data属性的设计:这里我们将一个按钮成为一个cell

1. `data-key_id`： 标识当前cell的规格id，唯一确定一种规格
2. `data-value_id`: 标识当前cell的具体规格id，唯一表示一种规格
3. `data-select`: 通过wxs实现一个函数，传入selected与disabled，来确定当前cell是可选还是已选还是禁用状态，
4. `data-x`: 用更简单的方式确定当前点击的cell是哪种类型的，只能确定相同x的是同一种规格，不能知道具体是哪种

5: `data-y`: 简单确定当前选中的cell是某种规格的一种，确定位置，x与y的设计可以更方便查找cell位置

再说明一下cell样式的确定也是通过wxs中的一个函数，传入seleted与disabled确定当前cell的样式。


**点击某中规格时候的处理**
``` javascript
handleClickSpecs(event) {
    const { key_id, value_id, select, x, y } = event.currentTarget.dataset
    if (select === 'disabled') {
      return;
    }
    
    if (select === 'selectable') {
      this.data.selected.forEach((item, index) => {
        if (item.x === x) {
          this.data.selected.splice(index, 1)
        }
      })
      this.data.selected.push({x, y, key_id, value_id});
      this.handleSelectOneOption(x, y, key_id, value_id);
      
    }

    if (select === 'active') {
      this.clearAllSelectedAndDisabled();
      this.data.selected.forEach((item, index) => {
        if (item.x === x && item.y === y) {
          this.data.selected.splice(index, 1);
        }
      })
      
      this.data.selected.forEach(item => {
        this.handleSelectOneOption(item.x, item.y, item.key_id, item.value_id);
      })      
    }

    this.setData({
      allSpecsList: this.data.allSpecsList,
      selected: this.data.selected
    })
  }
```
在点击具体的规格时候，我们会面临三种状态，默认的状态是可选的状态`selectable`，所以我们只用额外增加两种状态`active`已选状态和`disabled`禁用状态。通过event.currentTarget获取到点击的这个cell，然后通过其`dataset`属性获取到我们之前设计的一些辅助更简单获取当前cell属性的数据。

1. 如果是`disabled`的状态，直接返回，因为不可点。
2. 如果是`selectable`状态，当前cell可以点击，点击之后要对其他种类单元格做个遍历，将不匹配的置灰。并将当前选中的数据放入`selected`中保存起来方便后续使用，这里要强调的一点是在选择的时候要通过之前设置的x变量，遍历当前种类的规格，如果之前有选过，则删除它，因为一种规格只能选一个值。
3. 如果是`active`状态，点击则需要将其置为可选状态，这时候如果想要将某些置灰属性恢复可选，是不太好操作的，我们这里选择一种简单的方式，就是将当前点击的元素从`selected`中删除，然后遍历`selected`中的元素，将其重新“点击”一遍（当然这里使用代码来点击）。

这样就实现了基本的功能其中`handleSelectOneOption`的代码如下，它实现了点击一个cell，去置灰其他种类的规格

``` javascript
handleSelectOneOption(x, y, key_id, value_id) {
    
    this.data.allSpecsList[x].value_list[y].selected = true;
    this.data.allSpecsList[x].value_list.forEach((specs, index) => {
      if (index === y) {
        specs.selected = true;
      } else {
        specs.selected = false;
      }
    })
    const selectableMatchItems = this.data.selectable[key_id].selectableList[value_id].matchItems;
    this.data.allSpecsList.forEach((specsRow, index) => {
      if (index === x) {
        return;
      }
      specsRow.value_list.forEach(specs => {
        specs.disabled = false;
        if (specs.selected) {
          return;
        }
        const result = selectableMatchItems[specsRow.key_id].find(item => item.value_id === specs.value_id);
        if (!result) {
          specs.disabled = true;
        }
      })
    })
  }
```

通过x选项我们可以知道我们不需要遍历当前x这一行（也就是这一种属性），如果不是当前种类的属性，就拉过来遍历，如果不跟当前规格匹配，那就置灰。
有个细节要注意一下，在置灰过程中首先恢复了这种属性的置灰状态，这是为了防止之前置灰的效果影响当前置灰的结果。

在点击`active`状态的cell时候，我们也有一个细节操作，就是先将所有的置灰与选中全部恢复默认，再重新操作，也是为了防止之前状态影响我们现在的结果。

**清空所有置灰与选中的代码**

``` javascript
 clearAllSelectedAndDisabled() {
    this.data.allSpecsList.forEach(row => {
      row.value_list.forEach(specs => {
        specs.selected = false;
        specs.disabled = false;
      })
    })
  }
```

### 6.完善细节：联动上面的提示文字
我们看到实现的效果，在我们点击的时候，上方提示文字会有请选择xxx，当选完之后，会有已选xxx的文字效果，下面将代码展示出来。

``` javascript
getSelectedInfo() {
    let selectTips = '';

    if(this.data.selected.length === this.data.allSpecsList.length) {
      let selectedText = []
      this.data.allSpecsList.forEach(rowSpecs => {
        rowSpecs.value_list.forEach(specs => {
          if(specs.selected) {
            selectedText.push(specs.value)
          }
        })
      })
      selectTips = '已选：' + selectedText.join(',');
      const selectedItem = this.data.sku.getSelectable(selectedText.join('·'));
      this.setData({
        selectTips,
        selectedItem
      })
      return 
    }
    const selectedRow = this.data.selected.map(item => item.x);
    const unSelected = []
    this.data.allSpecsList.forEach((item, index) => {
      if(!selectedRow.includes(index)) {
        unSelected.push(item.key)
      }
    })
    selectTips = '请选择：' + unSelected.join(',');
    this.setData({
      selectTips
    })
  }
```

主要思路就是查看selected已选列表中的数量跟规格列表的种类数量是不是相同，如果相同那就全部选中了，然后通过这些选中的规格可以确定一个SKU，我们就能拿到这个SKU的库存，价格，图像等信息，这里我们把这个SKU存起来了，添加库显示等可以直接在页面添加相关数据。

如果已选属性数量跟规格种类数量不匹配，那就找有哪个规格的数据没选，将其提示在上方。

## 小结
当前SKU算法的实现可能不是最好的，但自己感觉不是很难理解，最好的效果是看思路然后亲自动手实现一次，有可能看着简单，但是动手时候会有很多意想不到的困难。

最后说明一下这篇文章主要是对慕课网学习中`7七月`老师布置作业的独立完成和思考，所有思路与代码也都是自己一整天思考和动手完成的，如果有小伙伴想一起加入成长，可以慕课来报`7七月`老师的从java后端到全栈课程的学习。