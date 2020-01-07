## 前言

接上一篇文章 [《「干货」用 Vue + Echarts 打造你的专属可视化界面（上）》](https://github.com/micherwa/blogs/blob/master/articles/2019/09-28%20%E3%80%8C%E5%B9%B2%E8%B4%A7%E3%80%8D%E7%94%A8%20Vue%20%2B%20Echarts%20%E6%89%93%E9%80%A0%E4%BD%A0%E7%9A%84%E4%B8%93%E5%B1%9E%E5%8F%AF%E8%A7%86%E5%8C%96%E7%95%8C%E9%9D%A2%EF%BC%88%E4%B8%8A%EF%BC%89.md)，今天着重介绍 `标记` 的用法，来实现下图中的效果。

![bg-20190928-01.png](https://github.com/micherwa/blogs/blob/master/images/2019/bg-20190928-01.png)

所用的 Echarts 的版本号为： v4.3。v-charts 的版本号为：v1.19.0。

标记的用法有很多，今天要介绍的场景有：折线图、柱状图、折线图 + 柱状图。

&nbsp;

## 折线图标记 —— symbol

上图中，折线的拐点处，一些 “小圆点”，被替换成了小图标。

要实现这样的效果，需要先理一下原始的需求：
- 每种标记，代表一种活动类型。
- 有一些活动会发生在某些时间点，或时间段内，需要在活动发生的日期上标注出该活动的类型。
- 当同一天有多个活动发生时，采用复合图标，并当展示 tooltip 时，显示当日的每一个活动的信息。
- tooltip的布局为：首先显示当前日期，中段展示各个活动的图标以及活动名称，最后展示指标名称和对应的数值。
- 没有活动的日期，拐点处与 tooltip 照常显示原先的样式。
所以，它的完整效果，应该是这样的：


![bg-20190928-12.png](https://github.com/micherwa/blogs/blob/master/images/2019/bg-20190928-12.png)

要实现这样的效果，需要思考以下几点：
- 如何通过日期的定向匹配，将活动的图标以“打点”的形式，定在折线拐点处。
- 图标的大小，要怎么设置？它需要区别于正常的拐点标识。
- tooltip 的样式要如何改写？还需要兼容没有活动的日期样式。

思路解析：

首先，为了做日期的定向匹配，需要设计的数据结构如下：

```
data: [
    {
        id: '1, 1, 3, 2',
        date: '2019-10-10',
        name: 'test-name1, test-name2, test-name3, test-name4'
    },
    ...
]
```

接下来的这个 核心 属性：`symbol` 是关键，它其实就是折线上的 `拐点`。

symbol 支持的标记类型有：circle、rect、roundRect、triangle、diamond、pin、arrow、none。默认情况下为 emptyCircle，也就是空心的圆。

它还支持链接的格式：`'image://http://xxx.xxx.xxx/a/b.png'`。此外，如果需要每个数据的图形不一样，可以设置为如下格式的回调函数：

```
(value: Array|number, params: Object) => string
```

其中第一个参数 value 为 data 中的数据值。第二个参数 params 是其它的数据项参数。

这些正是我们需要的。踩坑亲测：上述回调函数，只有在最新版的 `V4.3` 中才能正常使用，否则会报错。这也是为何，我在一开始就先强调了 Echarts 的版本问题。具体实现如下：

```
<ve-line ... :extend="chartExtend"></ve-line>
...

// mock 包含标注的数据结构
dataList: [
    {
        id: '1, 1, 3, 2',
        date: '2019-10-10',
        name: 'test-name1, test-name2, test-name3, test-name4'
    },
    ...
    {
        id: '1',
        date: '2019-10-17',
        name: 'test-name1'
    }
],
...

setChartExtend () {
    this.chartExtend = {
        series: (v) => {
            Array.from(v).forEach((e, idx) => {
                e.symbol = (value, params) => {
                    return getSymbolIcon(params.name, dataList);
                };
                e.symbolSize = (value, params) => {
                    return getSymbolSize(params.name, dataList);
                };
            });
            return v;
        },
        ...
    };
},

getSymbolIcon (date, dataList) {
    const defaultSymbol = 'circle';

    if (!dataList || dataList.length === 0) {
        return defaultSymbol;
    }

    // 通过日期匹配，找到对应的标注对象
    const dataItem = dataList.find(item => item.date === date);

    const iconUrl = getSymbolUrl(dataItem.id);
    return iconUrl ? iconUrl : defaultSymbol;
},

getSymbolSize (date, dataList) {
    if (!dataList || dataList.length === 0) {
        return 4;
    }

    // 通过日期匹配，找到对应的标注对象
    const dataItem = dataList.find(item => item.date === date);

    return dataItem ? 15 : 4;
},

getSymbolUrl (id) {
    // 这里需要额外先做一层准备工作：将图标按 id 对应图标进行命名，然后传到自家的cdn上
    // 命名可以像这样：symbol-icon-1.jpg、symbol-icon-2.jpg 等等
    // 这里拿到标注的id，拼上链接返回即可
    // 形如：image://http://xxx.xxx.com/symbol-icon-1.jpg
    // 遇到多个 id 的情况，可以多加一个复合图标来处理，id 可以定为 0
}
```

最后的一个问题，如何改写 tooltip 的样式问题，以做好兼容呢？

之前说到，tooltip 的布局分为三块：日期、标注信息、具体数值。那么我们就以此，来重新绘制 tooltip。


![bg-20190928-13.png](https://github.com/micherwa/blogs/blob/master/images/2019/bg-20190928-13.png)

tooltip 支持 formatter 回调函数，它的返回值类型是 Sting。

```
// 回调函数格式
(params: Object|Array, ticket: string, callback: (ticket: string, html: string)) => string
```

日期的信息，可以通过 params[0].axisValue 来获取。

获取标注信息的方法，与上述获取图标的思路类似，只是这里需要展示具体的标注类型和名称。

具体的数值，可以通过 params 中的 marker，seriesName，value 等属性获得。具体实现如下：

```
setChartExtend () {
    this.chartExtend = {
        series: (v) => {
            ...
        },
        tooltip: {
            formatter: (params) => {
                return getTooltipResult(params, dataList);
            }
        }
    };
},

getTooltipResult (params, dataList) {
    const dateResult = params[0].axisValue;
    // 获取原版 tooltip 的渲染结构
    const originalResultObj = getOriginalTooltipResult(params);

    if (!dataList || dataList.length === 0) {
        return dateResult + originalResultObj.strResult;
    }

    const dataItem = dataList.find(item => item.date === date);

    if (dataItem) {
        return dateResult +  getSymbolResult(dataItem, originalResultObj.strResult);
    }

    return dateResult + originalResultObj.strResult;
},

getOriginalTooltipResult (params) {
    let result = '';

    params.forEach((param, idx) => {
        // value 会因为 seriesType 的不同，类型也会有不同
        let value = Object.prototype.toString.call(param.value) === '[object Array]' ? param.value[1] : param.value;

        const str = `${param.marker}${param.seriesName}: ${ value }<br>`;
        result += str;
    });

    return {
        strResult: result
    };
},

getSymbolResult (dataItem, originalResult) {
    // 将 dataItem 的 id 转为数组的形式，循环渲染输出图标与名称的组合
    const dataIds = dataItem.id.split(',');
    const dataNames = dataItem.name.split(',');

    dataIds.forEach ((id, idx) => {
        // 通过 id 换取 图标的链接
        const iconUrl = ...;

        // 仿照 param.marker 的 style 写法，渲染图标样式
        const str = `<img src="${iconUrl}" width="11" height="11" style="display: inline-block; margin-right: 4px; margin-left: -1px;">${dataNames[idx]}<br>`;

        result += str;
    });

    return result + originalResult;
}
```

或许有同学会问 getOriginalTooltipResult 方法返回的值，里面只有一个 strResult，为何要设计为对象？

其实是为了方便扩展。例如，可以在日期的后面跟上下方具体数据的值的总计。那就需要通过 getOriginalTooltipResult 方法里的 params 循环，计算出 total，在配合上样式，生成一个 strTotal。

```
getOriginalTooltipResult (params) {
    ...

    return {
        strTotal: strTotal,
        strResult: result
    };
}
```

此外，在实际的业务中，还可能会出现某些线的数值是百分比。那就需要再对 getOriginalTooltipResult 方法做扩展，比如传入一个 options 对象：

```
getOriginalTooltipResult = (params, options = { isLinePercent: false, isShowTotal: false }) {
    ...
}
```

至此，折线图标记的渲染，就能完美地呈现了。

&nbsp;

## 柱状图标记 —— markPoint

很尴尬的一点是：柱状图没有 symbol 属性。也就意味着上面的折线图的那一套，在柱状图中玩不转了。


![bg-20190928-14.jpg](https://github.com/micherwa/blogs/blob/master/images/2019/bg-20190928-14.jpg)

没办法，只能从头查文档，继续找资料。经过一番“摸爬滚打”，终于发现了 markPoint 这个属性。在 markPoint 这个对象里面，可以设置 symbol，这样的话，那么之前搞出来的那一套就没有白费呀？！？

需要注意的是，markPoint 的默认 symbol 为 'pin'，就是一个气泡的图标。另外，想要让 markPoint 的标记出现，就必须设置它的 data 属性。我们需要设置 data 里的这样几个属性：

```
data: [
    {
        symbol: '...', // 设置标记的图标链接
        symbolSize: 15, // 设置标记的大小
        coord: [index, 0], // x 轴的第 index 个上，打标记
        symbolOffset: [0, 0] // 将标记定位在 x 轴上
    },
    ...
]
```

具体的实现代码如下：

```
<ve-histogram :data="chartData" :extend="chartExtend"></ve-histogram>
...

// mock 包含标注的数据结构
dataList: [
    {
        id: '1, 1, 3, 2',
        date: '2019-10-10',
        name: 'test-name1, test-name2, test-name3, test-name4'
    },
    ...
    {
        id: '1',
        date: '2019-10-17',
        name: 'test-name1'
    }
],
...

setChartExtend () {
    this.chartExtend = {
        series: (v) => {
            Array.from(v).forEach((e, idx) => {
                e.markPoint = {
                    data: getMarkPointData(this.chartData.rows, dataList)
                };
            });
        },
        ...
    };
},

getMarkPointData (rows, dataList) {
    const results = [];

    rows.forEach((row, index) => {
        // 通过日期匹配，找到对应的标注对象
        const dataItem = dataList.find(item => item.date === row.date);
        if (dataItem) {
            results.push({
                symbol: getSymbolUrl(dataItem.id),
                symbolSize: 15,
                coord: [index, 0],
                symbolOffset: [0, 0]
            });
        }
    });

    return results;
},

getSymbolUrl (id) {
    // 这里需要额外先做一层准备工作：将图标按 id 对应图标进行命名，然后传到自家的cdn上
    // 命名可以像这样：symbol-icon-1.jpg、symbol-icon-2.jpg 等等
    // 这里拿到标注的id，拼上链接返回即可
    // 形如：image://http://xxx.xxx.com/symbol-icon-1.jpg
    // 遇到多个 id 的情况，可以多加一个复合图标来处理，id 可以定为 0
}
```

因为 markPoint 在设置 data 时，取不到日期的数据，所以就需要用到 chartData 中的 rows 了。

在 rows 的循环中，如果匹配到当天需要打标记，则往结果数组中存入刚才预设的数据结构，最终返回给 markPoint 的 data，渲染展现。效果如下：


![bg-20190928-15.png](https://github.com/micherwa/blogs/blob/master/images/2019/bg-20190928-15.png)

tooltip 的实现方法，不受图表的类型影响，是可以通用的，故此处不再赘述。

另外，有的时候，会遇到需要处理柱状图是否堆叠的效果。这会影响 symbolOffset 的定位，为了美观，可以这样处理：对于非堆叠的项，将之往右偏移 50%，将标记居中展示，即 `symbolOffset : ['50%', 0]`。

&nbsp;

## 折线图 + 柱状图

最后来个 **组合拳**，折线图与柱状图的复合型图表结构，像下面这样：


![bg-20190928-16.png](https://github.com/micherwa/blogs/blob/master/images/2019/bg-20190928-16.png)

从代码实现上，我们当然可以给每一根符合条件的柱子，和折线的拐点，都打上标记。但在界面设计上，为了美观，我们选择只给折线的拐点打上标记，并且忽略了虚线的拐点。

这样的设计初衷是：标记，只是为了给人提个醒，是为了告诉查阅者，这一天因为发生了某些特殊事件，而导致数据发生了较为明显的变化。所以，由此得到的结论是：每天只要出现一个标记就够了。

具体的实现，其实很简单，只需要在渲染时判断 series 的 type 即可：

```
setChartExtend () {
    this.chartExtend = {
        series: (v) => {
            Array.from(v).forEach((e, idx) => {
                if (e.type === 'bar') {
                    // 设置柱状图 markPoint 的方法
                    ...
                }

                if (e.type === 'line') {
                    // 设置折线图 symbol、symbolSize 的方法
                    ...
                }
            });
        },
        ...
    };
}
```

&nbsp;

## 总结

Echarts 可以实现的效果有很多，本篇涉及其中 “标记” 的渲染。在折线图的拐点处，用 symbol 做了匹配化的处理。在柱状图中，因为没有直接的 symbol，转而使用 markPoint 来实现，采用将标记定位在某个维度上的做法。效果都挺不错的。

不过，在后续的使用中，发现了另一个尴尬的情况：`在折线图中，当点击图例中的某一项，使其数据隐藏，然后再次点击重新渲染后，发现 symbol 的自定义图标不显示了`。

我查了很久，还是没找到有用的信息。怀疑是个渲染的 bug，所以给 Echarts 提了 issue，希望能得到解决吧。也欢迎大家在留言区中，共同探讨相关的问题，感谢！


&nbsp;

**PS：欢迎关注我的公众号 “超哥前端小栈”，交流更多的想法与技术。**

![wechat qrCode](https://github.com/micherwa/blogs/blob/master/images/wechat_qrCode.jpg)

&nbsp;

*Posted on 2019-10-22*