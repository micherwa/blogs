![bg-20190228-01.jpg](https://github.com/micherwa/blogs/blob/master/images/2019/bg-20190228-01.jpg)

## 前言

这是前端面试题系列的第 7 篇，你可能错过了前面的篇章，可以在这里找到：

- [理解函数的柯里化](https://github.com/micherwa/blogs/blob/master/articles/2019/02-14%20%E3%80%8C%E5%89%8D%E7%AB%AF%E9%9D%A2%E8%AF%95%E9%A2%98%E7%B3%BB%E5%88%976%E3%80%8D%E7%90%86%E8%A7%A3%E5%87%BD%E6%95%B0%E7%9A%84%E6%9F%AF%E9%87%8C%E5%8C%96.md)
- [ES6 中箭头函数的用法](https://github.com/micherwa/blogs/blob/master/articles/2019/02-10%20%E3%80%8C%E5%89%8D%E7%AB%AF%E9%9D%A2%E8%AF%95%E9%A2%98%E7%B3%BB%E5%88%975%E3%80%8DES6%20%E4%B8%AD%E7%AE%AD%E5%A4%B4%E5%87%BD%E6%95%B0%E7%9A%84%E7%94%A8%E6%B3%95.md)
- [this 的原理以及用法](https://github.com/micherwa/blogs/blob/master/articles/2019/01-16%20%E3%80%8C%E5%89%8D%E7%AB%AF%E9%9D%A2%E8%AF%95%E9%A2%98%E7%B3%BB%E5%88%974%E3%80%8Dthis%E7%9A%84%E5%8E%9F%E7%90%86%E4%BB%A5%E5%8F%8A%E7%94%A8%E6%B3%95.md)
- [伪类与伪元素的区别及实战](https://github.com/micherwa/blogs/blob/master/articles/2019/01-04%20%E3%80%8C%E5%89%8D%E7%AB%AF%E9%9D%A2%E8%AF%95%E9%A2%98%E7%B3%BB%E5%88%973%E3%80%8D%E4%BC%AA%E7%B1%BB%E4%B8%8E%E4%BC%AA%E5%85%83%E7%B4%A0%E7%9A%84%E5%8C%BA%E5%88%AB%E5%8F%8A%E5%AE%9E%E6%88%98.md)
- [如何实现一个圣杯布局？](https://github.com/micherwa/blogs/blob/master/articles/2018/12-23%20%E3%80%8C%E5%89%8D%E7%AB%AF%E9%9D%A2%E8%AF%95%E9%A2%98%E7%B3%BB%E5%88%972%E3%80%8D%E5%A6%82%E4%BD%95%E5%AE%9E%E7%8E%B0%E4%B8%80%E4%B8%AA%E5%9C%A3%E6%9D%AF%E5%B8%83%E5%B1%80%EF%BC%9F.md)
- [今日头条 面试题和思路解析](https://github.com/micherwa/blogs/blob/master/articles/2018/12-19%20%E3%80%8C%E5%89%8D%E7%AB%AF%E9%9D%A2%E8%AF%95%E9%A2%98%E7%B3%BB%E5%88%971%E3%80%8D%E4%BB%8A%E6%97%A5%E5%A4%B4%E6%9D%A1%20%E9%9D%A2%E8%AF%95%E9%A2%98%E5%92%8C%E6%80%9D%E8%B7%AF%E8%A7%A3%E6%9E%90.md)

最近，小伙伴L 在温习 《JavaScript高级程序设计》中的 `事件` 这一章节时，产生了困惑。

他问了我这样几个问题：

- 了解事件流的顺序，对日常的工作有什么帮助么？
- 在 vue 的文档中，有一个修饰符 **native** ，把它用 `.` 的形式 连结在事件之后，就可以监听原生事件了。它的背后有什么原理？
- 事件的 event 对象中，有好多的属性和方法，该如何使用？

浏览器中的事件机制，也经常在面试中被提及。所以这回，我们共同探讨了这些问题，并最终整理成文，希望帮到有需要的同学。

&nbsp;

## 事件流的概念

先从概念说起，DOM 事件流分为三个阶段：`捕获阶段`、`目标阶段`、`冒泡阶段`。先调用捕获阶段的处理函数，其次调用目标阶段的处理函数，最后调用冒泡阶段的处理函数。


![bg-20190228-02.png](https://github.com/micherwa/blogs/blob/master/images/2019/bg-20190228-02.png)

网景公司提出了 `事件捕获` 的事件流。这就好比采矿的小游戏，每次都会从地面开始一路往下，抛出抓斗，捕获矿石。在上图中就是，某个 div 元素触发了某个事件，最先得到通知的是 window，然后是 document，依次往下，直到真正触发事件的那个目标元素 div 为止。

而 `事件冒泡` 则是由微软提出的，与之顺序相反。还是刚才的采矿小游戏，命中目标后，抓斗再沿路收回，直到冒出地面。在上图中就是，事件会从目标元素 div 开始依次往上，直到 window 对象为止。

w3c 为了制定统一的标准，采取了折中的方式：`先捕获在冒泡`。同一个 DOM 元素可以注册多个同类型的事件，通过 addEventListener 和 removeEventListener 进行管理。addEventListener 的第三个参数，就是为了捕获和冒泡准备的。

`注册事件`(addEventListener) 有三个参数，分别为："事件名称", "事件回调", "捕获/冒泡"(布尔型，true代表捕获事件，false代表冒泡事件)。

```
target.addEventListener(type, listener[, useCapture]);
```
- type 表示事件类型的字符串。
- listener 是一个实现了 EventListener 接口的对象，或者是一个函数。当所监听的事件类型触发时，会接收到一个事件通知对象（实现了 Event 接口的对象）。
- capture 表示 listener 会在该类型的事件捕获阶段，传播到该 EventTarget 时触发，它是一个 Boolean 值。

`解除事件`(removeEventListener) 也有三个参数，分别为："事件名称", "事件回调", "捕获/冒泡"(Boolean 值，这个必须和注册事件时的类型一致)。

```
target.removeEventListener(type, listener[, useCapture]);
```
要想注册过的事件能够被解除，必须将回调函数保存起来，否则无法解除。例如这样：
```
const btn = document.getElementById("test");

//将回调存储在变量中
const fn = function(e){
    alert("ok");
};

//绑定
btn.addEventListener("click", fn, false);

//解除
btn.removeEventListener("click", fn, false);
```

&nbsp;

## 事件捕获和冒泡的5个注意点

当有多层交互嵌套时，事件捕获和冒泡的先后顺序，似乎不是那么好理解。接下来，将分 5 种情况讨论它们的顺序，以及如何规避意外情况的发生。

#### 1.在外层 div 注册事件，点击内层 div 来触发事件时，捕获事件总是要比冒泡事件先触发(与代码顺序无关)

假设，有这样的 html 结构：

```
<div id="test" class="test">
   <div id="testInner" class="test-inner"></div>
</div>
```

然后，我们在外层 div 上注册两个 click 事件，分别是捕获事件和冒泡事件，代码如下：

```
const btn = document.getElementById("test");

//捕获事件
btn.addEventListener("click", function(e){
    alert("capture is ok");
}, true);

//冒泡事件
btn.addEventListener("click", function(e){
    alert("bubble is ok");
}, false);
```

点击内层的 div，先弹出 capture is ok，后弹出 bubble is ok。只有当真正触发事件的 DOM 元素是内层的时候，外层 DOM 元素才有机会模拟捕获事件和冒泡事件。

#### 2.当在触发事件的 DOM 元素上注册事件时，哪个先注册，就先执行哪个

html 结构同上，js 代码如下：
```
const btnInner = document.getElementById("testInner");

//冒泡事件
btnInner.addEventListener("click", function(e){
    alert("bubble is ok");
}, false);

//捕获事件
btnInner.addEventListener("click", function(e){
    alert("capture is ok");
}, true);

```

本例中，冒泡事件先注册，所以先执行。所以，点击内层 div，先弹出 `bubble is ok`，再弹出 `capture is ok`。

#### 3.当外层 div 和内层 div 同时注册了捕获事件时，点击内层 div 时，外层 div 的事件一定会先触发

js 代码如下：

```
const btn = document.getElementById("test");
const btnInner = document.getElementById("testInner");

btnInner.addEventListener("click", function(e){
    alert("inner capture is ok");
}, true);

btn.addEventListener("click", function(e){
    alert("outer capture is ok");
}, true);
```
虽然外层 div 的事件注册在后面，但会先触发。所以，结果是先弹出 `outer capture is ok`，再弹出 `inner capture is ok`。

#### 4.同理，当外层 div 和内层 div 都同时注册了冒泡事件，点击内层 div 时，一定是内层 div 事件先触发。

```
const btn = document.getElementById("test");
const btnInner = document.getElementById("testInner");

btn.addEventListener("click", function(e){
    alert("outer bubble is ok");
}, false);

btnInner.addEventListener("click", function(e){
    alert("inner bubble is ok");
}, false);
```
先弹出 `inner bubble is ok`，再弹出 `outer bubble is ok`。

#### 5.阻止事件的派发

通常情况下，我们都希望点击某个 div 时，就只触发自己的事件回调。比如，明明点击的是内层 div，但是外层 div 的事件也触发了，这是就不是我们想要的了。这时，就需要阻止事件的派发。

事件触发时，会默认传入一个 event 对象，这个 event 对象上有一个方法：`stopPropagation`。MDN 上的解释是：**阻止 捕获 和 冒泡 阶段中，当前事件的进一步传播**。所以，通过此方法，让外层 div 接收不到事件，自然也就不会触发了。

```
btnInner.addEventListener("click", function(e){
    //阻止冒泡
    e.stopPropagation();
    alert("inner bubble is ok");
}, false);
```

&nbsp;

## 事件代理

我们经常会遇到，要监听列表中多项 li 的情况，假设我们有一个列表如下：

```
<ul id="list">
    <li id="item1">item1</li>
    <li id="item2">item2</li>
    <li id="item3">item3</li>
    <li id="item4">item4</li>
</ul>
```

如果我们要实现以下功能：当鼠标点击某一 li 时，输出该 li 的内容，我们通常的写法是这样的：
```
window.onload=function(){
    const ulNode = document.getElementById("list");
    const liNodes = ulNode.children;
    for(var i=0; i<liNodes.length; i++){
        liNodes[i].addEventListener('click',function(e){
            console.log(e.target.innerHTML);
        }, false);
    }
}
```

在传统的事件处理中，我们可能会按照需要，为每一个元素添加或者删除事件处理器。然而，事件处理器将有可能导致内存泄露，或者性能下降，用得越多这种风险就越大。JavaScript 的事件代理，则是一种简单的技巧。

#### 用法及原理

事件代理，用到了在 JavaSciprt 事件中的两个特性：事件冒泡 和 目标元素。使用事件代理，我们可以把事件处理器添加到一个元素上，等待一个事件从它的子级元素里冒泡上来，并且可以得知这个事件是从哪个元素开始的。

改进后的 js 代码如下：
```
window.onload=function(){
    const ulNode=document.getElementById("list");
    ulNode.addEventListener('click', function(e) {
        /*判断目标事件是否为li*/
        if(e.target && e.target.nodeName.toUpperCase()=="LI"){
            console.log(e.target.innerHTML);
        }
    }, false);
};
```

&nbsp;

## 一些常用技巧

回到文章开头的问题：了解事件流的顺序，对日常的工作有什么帮助呢？我总结了以下几个注意点。

#### 1. 阻止默认事件

比如 href 的链接跳转，submit 的表单提交等。可以在方法的最后，加上一行 `return false;`。它会阻止通过 on 的方式绑定的事件的默认事件。
```
ele.onclick = function() {
    ……
    // 通过返回 false 值，阻止默认事件行为
    return false;
}
```
另外，重写 onclick 会覆盖之前的属性，所以解绑事件可以这么写：

```
// 解绑事件，将 onlick 属性设为 null 即可
ele.onclick = null;
```

#### 2. stopPropagation 和 stopImmediatePropagation

前面说过 stopPropagation 的定义是：终止事件在传播过程的捕获、目标处理或起泡阶段进一步传播。事件不再被分派到其他节点上。
```
// 事件捕获到 ele 元素后，就不再向下传播了
ele.addEventListener('click', function (event) {
  event.stopPropagation();
}, true);

// 事件冒泡到 ele 元素后，就不再向上传播了
ele.addEventListener('click', function (event) {
  event.stopPropagation();
}, false);
```

但是，stopPropagation 只会阻止当前元素 `同类型的` 事件冒泡或捕获的传播，并不会阻止该元素上 `其他类型` 事件的监听。以 click 事件为例：
```
ele.addEventListener('click', function (event) {
  event.stopPropagation();
  console.log(1);
});

ele.addEventListener('click', function(event) {
  // 仍然可以触发
  console.log(2);
});
```

如果想禁用之后所有的 click 事件，就要用到 stopImmediatePropagation 了。但是，需要注意的是，stopImmediatePropagation 只会禁用之后注册的同类型的监听事件。就比如阻止了之后的 click 事件监听函数，但别的事件类型如 mousedown、dblclick 之类，还是可以监听到的。
```
ele.addEventListener('click', function (event) {
    event.stopImmediatePropagation();
    console.log(1);
});

ele.addEventListener('click', function(event) {
    // 不会触发
    console.log(2);
});

ele.addEventListener('mousedown', function(event) {
    // 会触发
    console.log(3);
});
```

#### 3. jquery 中的 return false;

jquery 中的 on 是事件冒泡。当用 return false; 阻止浏览器的默认行为时，会做下面这 3 件事：

- event.preventDefault();
- event.stopPropagation();
- 停止回调函数执行并立即返回。

这 3 件事中，只有 preventDefault 是用来阻止默认行为的。除非你还想阻止事件冒泡，否则直接用 return false; 会埋下隐患。

#### 4. angular 中的 $event

angular 是个包罗万象的框架，似乎学完它的一整套之后，就能玩转世界了。它加工封装了许多原生的东西，其中就包括了 event，只是前面需要加一个 $，表示这是 angular 中的特有对象。
```
// template
<div>
    <button (click)="doSomething($event)">Click me</button>
</div>

// js
doSomething($event: Event) {
    $event.stopPropagation();
    ...
}
```
$event 在这里作为一个变量，`显式地` 传入回调函数，之后就可以将 $event 当做原生的事件对象来用了。

#### 5. vue 中的 native 修饰符

在 vue 的自定义组件中绑定原生事件，需要用到修饰符 native。

那是因为，我们的自定义组件，最终会渲染成原生的 html 标签，而非类似于 这样的自定义组件。如果想让一个普通的 html 标签触发事件，那就需要对它做事件监听(addEventListener)。修饰符 native 的作用就在这里，它可以在背后帮我们绑定了原生事件，进行监听。

一个常用的场景是，配合 element-ui 做登录界面时，输完账号密码，想按一下回车就能登录。就可以像下面这样用修饰符：
```
<el-input
    class="input"
    v-model="password" type="password"
    @keyup.enter.native="handleSubmit">
</el-input>
```

el-input 就是自定义组件，而 keyup 就是原生事件，需要用 native 修饰符进行绑定才能监听到。

#### 6. react 中的合成事件

想要在 react 的事件回调中使用 event 对象，会产生困扰，会发现不少原生的属性都是 null。

那是因为在 react 中的事件，其实是合成事件（SyntheticEvent），并不是浏览器的原生事件，但它也符合 w3c 规范。

举一个简单的例子，我们要实现一个组件，它有一个按钮，点击按钮后会显示一张图片，点击这张图片之外的任意区域，可以隐藏这张图片，但是点击该图片本身时，不会隐藏。代码如下：
```
class ShowImg extends Component {
    constructor(props) {
        super(props);
        this.state = {
          active: false
        };
    }

    componentDidMount() {
        document.addEventListener('click', this.hideImg.bind(this));
    }

    componentWillUnmount() {
        document.removeEventListener('click', this.hideImg);
    }

    hideImg () {
        this.setState({ active: false });
    }

    handleClickBtn() {
        this.setState({ active: !this.state.active });
    }

    handleClickImg (e) {
        e.stopPropagation();
    }

    render() {
        return (
            <div className="img-wrapper">
                <button
                    className="showImgBtn"
                    onClick={this.handleClickBtn.bind(this)}>
                    显示图片
                </button>
                <div
                    className="img"
                    style={{ display: this.state.active ? 'block' : 'none' }}
                    onClick={this.handleClickImg.bind(this)}>
                    <img src="@/assets/avatar.jpg" >
                </div>
            </div>
        );
    }
}
```

按照之前说的原生事件机制，我们会错误地认为通过：
```
handleClickImg (e) {
    e.stopPropagation();
}
```

就可以阻止事件的派发了，但其实没法这么做。想要解决这个问题，当然也不复杂，就把 react 的事件和原生事件分开即可。
```
componentDidMount() {
    document.addEventListener('click', this.hideImg.bind(this));

    document.addEventListener('click', this.imgStopPropagation.bind(this));
}

componentWillUnmount() {
    document.removeEventListener('click', this.hideImg);

    document.removeEventListener('click', this.imgStopPropagation);
}

hideImg () {
    this.setState({ active: false });
}

imgStopPropagation (e) {
    e.stopPropagation();
}
```

#### 7. 事件对象 event

当对一个元素进行事件监听的时候，它的回调函数里就会默认传递一个参数 event，它是一个对象，包含了许多属性。我列出了一些比较常用的属性：
- event.target：指的是触发事件的那个节点，也就是事件最初发生的节点。
- event.target.matches：可以对关键节点进行匹配，来执行相应操作。
- event.currentTarget：指的是正在执行的监听函数的那个节点。
- event.isTrusted：表示事件是否是真实用户触发。
- event.preventDefault()：取消事件的默认行为。
- event.stopPropagation()：阻止事件的派发（包括了捕获和冒泡）。
- event.stopImmediatePropagation()：阻止同一个事件的其他监听函数被调用。

&nbsp;

## 总结

事件机制在浏览器中非常有用，所有用户的交互型操作，都依赖于它。现代 JavaScript 框架应用中，我们也都离不开与原生事件的交互。

所以，在理解了事件流的概念，清楚了事件捕获与冒泡的顺序，掌握了一些原生事件的技巧之后，相信下次再遇到坑的时候，可以少走一些弯路了。


&nbsp;

**PS：欢迎关注我的公众号 “超哥前端小栈”，交流更多的想法与技术。**

![wechat qrCode](https://github.com/micherwa/blogs/blob/master/images/wechat_qrCode.jpg)

&nbsp;

*Posted on 2019-02-18*