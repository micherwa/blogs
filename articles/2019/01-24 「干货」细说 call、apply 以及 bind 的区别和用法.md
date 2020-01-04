
![bg-20190124-01.jpg](https://github.com/micherwa/blogs/blob/master/images/2019/bg-20190124-01.jpg)

## 前言

上一篇文章 [《「前端面试题系列4」this 的原理以及用法》](https://github.com/micherwa/blogs/blob/master/articles/2019/01-16%20%E3%80%8C%E5%89%8D%E7%AB%AF%E9%9D%A2%E8%AF%95%E9%A2%98%E7%B3%BB%E5%88%974%E3%80%8Dthis%E7%9A%84%E5%8E%9F%E7%90%86%E4%BB%A5%E5%8F%8A%E7%94%A8%E6%B3%95.md) 中，提到了 call 和 apply。

它们最主要的作用，是改变 this 的指向。在平时的工作中，除了在写一些基础类，或者公用库方法的时候会用到它们，其他时候 call 和 apply 的应用场景并不多。

不过，突然遇到的时候，需要想一下才能转过弯来。所以今天，就让我们好好地探究一下，这两个方法的区别以及一些妙用。最后，还会介绍与之用法相似的 bind 的方法。

&nbsp;

## call 和 apply 的共同点

它们的共同点是，都能够**改变函数执行时的上下文**，将一个对象的方法交给另一个对象来执行，并且是立即执行的。

为何要改变执行上下文？举一个生活中的小例子：平时没时间做饭的我，周末想给孩子炖个腌笃鲜尝尝。但是没有适合的锅，而我又不想出去买。所以就问邻居借了一个锅来用，这样既达到了目的，又节省了开支，一举两得。

改变执行上下文也是一样的，A 对象有一个方法，而 B 对象因为某种原因，也需要用到同样的方法，那么这时候我们是单独为 B 对象扩展一个方法呢，还是借用一下 A 对象的方法呢？当然是借用 A 对象的啦，既完成了需求，又减少了内存的占用。

另外，它们的写法也很类似，**调用 call 和 apply 的对象，必须是一个函数 Function**。接下来，就会说到具体的写法，那也是它们区别的主要体现。

&nbsp;

## call 和 apply 的区别

它们的区别，主要体现在参数的写法上。先来看一下它们各自的具体写法。

#### call 的写法

```
Function.call(obj,[param1[,param2[,…[,paramN]]]])
```

需要注意以下几点：

- 调用 call 的对象，必须是个函数 Function。
- call 的第一个参数，是一个对象。 Function 的调用者，将会指向这个对象。如果不传，则默认为全局对象 window。
- 第二个参数开始，可以接收任意个参数。每个参数会映射到相应位置的 Function 的参数上。但是如果将所有的参数作为数组传入，它们会作为一个整体映射到 Function 对应的第一个参数上，之后参数都为空。

```
function func (a,b,c) {}

func.call(obj, 1,2,3)
// func 接收到的参数实际上是 1,2,3

func.call(obj, [1,2,3])
// func 接收到的参数实际上是 [1,2,3],undefined,undefined
```

#### apply 的写法
```
Function.apply(obj[,argArray])
```

需要注意的是：
- 它的调用者必须是函数 Function，并且只接收两个参数，第一个参数的规则与 call 一致。
- 第二个参数，必须是数组或者类数组，它们会被转换成类数组，传入 Function 中，并且会被映射到 Function 对应的参数上。这也是 call 和 apply 之间，很重要的一个区别。

```
func.apply(obj, [1,2,3])
// func 接收到的参数实际上是 1,2,3

func.apply(obj, {
    0: 1,
    1: 2,
    2: 3,
    length: 3
})
// func 接收到的参数实际上是 1,2,3
```

#### 什么是类数组？

先说数组，这我们都熟悉。它的特征有：可以通过角标调用，如 array[0]；具有长度属性length；可以通过 for 循环或forEach方法，进行遍历。

那么，类数组是什么呢？顾名思义，就是**具备与数组特征类似的对象**。比如，下面的这个对象，就是一个类数组。
```
let arrayLike = {
    0: 1,
    1: 2,
    2: 3,
    length: 3
};
```

类数组 arrayLike 可以通过角标进行调用，具有length属性，同时也可以通过 for 循环进行遍历。

类数组，还是比较常用的，只是我们平时可能没注意到。比如，我们获取 DOM 节点的方法，返回的就是一个类数组。再比如，在一个方法中使用 arguments 获取到的所有参数，也是一个类数组。

但是需要注意的是：**类数组无法使用 forEach、splice、push 等数组原型链上的方法**，毕竟它不是真正的数组。

&nbsp;

## call 和 apply 的用途

下面会分别列举 call 和 apply 的一些使用场景。声明：例子中没有哪个场景是必须用 call 或者必须用 apply 的，只是个人习惯这么用而已。

#### call 的使用场景

**1、对象的继承**。如下面这个例子：

```
function superClass () {
    this.a = 1;
    this.print = function () {
        console.log(this.a);
    }
}

function subClass () {
    superClass.call(this);
    this.print();
}

subClass();
// 1
```

subClass 通过 call 方法，继承了 superClass 的 print 方法和 a 变量。此外，subClass 还可以扩展自己的其他方法。

**2、借用方法**。还记得刚才的类数组么？如果它想使用 Array 原型链上的方法，可以这样：
```
let domNodes = Array.prototype.slice.call(document.getElementsByTagName("*"));
```

这样，domNodes 就可以应用 Array 下的所有方法了。

#### apply 的一些妙用

**1、Math.max**。用它来获取数组中最大的一项。

```
let max = Math.max.apply(null, array);
```

同理，要获取数组中最小的一项，可以这样：

```
let min = Math.min.apply(null, array);
```

**2、实现两个数组合并**。在 ES6 的扩展运算符出现之前，我们可以用 Array.prototype.push来实现。
```
let arr1 = [1, 2, 3];
let arr2 = [4, 5, 6];

Array.prototype.push.apply(arr1, arr2);
console.log(arr1); // [1, 2, 3, 4, 5, 6]
```

&nbsp;

## bind 的使用
最后来说说 bind。在 MDN 上的解释是：bind() 方法创建一个新的函数，在调用时设置 this 关键字为提供的值。并在调用新函数时，将给定参数列表作为原函数的参数序列的前若干项。

它的语法如下：
```
Function.bind(thisArg[, arg1[, arg2[, ...]]])
```

bind 方法 与 apply 和 call 比较类似，也能改变函数体内的 this 指向。不同的是，**bind 方法的返回值是函数，并且需要稍后调用，才会执行**。而 apply 和 call 则是立即调用。

来看下面这个例子：
```
function add (a, b) {
    return a + b;
}

function sub (a, b) {
    return a - b;
}

add.bind(sub, 5, 3); // 这时，并不会返回 8
add.bind(sub, 5, 3)(); // 调用后，返回 8
```
如果 bind 的第一个参数是 null 或者 undefined，this 就指向全局对象 window。

&nbsp;

## 总结

call 和 apply 的主要作用，是改变对象的执行上下文，并且是立即执行的。它们在参数上的写法略有区别。

bind 也能改变对象的执行上下文，它与 call 和 apply 不同的是，返回值是一个函数，并且需要稍后再调用一下，才会执行。

最后，分享一个在知乎上看到的，关于 call 和 apply 的便捷记忆法：

> 猫吃鱼，狗吃肉，奥特曼打小怪兽。
>
> 有天狗想吃鱼了
>
> 猫.吃鱼.call(狗，鱼)
>
> 狗就吃到鱼了
>
> 猫成精了，想打怪兽
>
> 奥特曼.打小怪兽.call(猫，小怪兽)
>
> 猫也可以打小怪兽了


&nbsp;

**PS：欢迎关注我的公众号 “超哥前端小栈”，交流更多的想法与技术。**

![wechat qrCode](https://github.com/micherwa/blogs/blob/master/images/wechat_qrCode.jpg)

&nbsp;

*Posted on 2019-01-24*
