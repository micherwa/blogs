## 前言

上一篇文章 [「前端面试题系列8」数组去重(10 种浓缩版)](https://github.com/micherwa/blogs/blob/master/articles/2019/02-28%20%E3%80%8C%E5%89%8D%E7%AB%AF%E9%9D%A2%E8%AF%95%E9%A2%98%E7%B3%BB%E5%88%978%E3%80%8D%E6%95%B0%E7%BB%84%E5%8E%BB%E9%87%8D(10%20%E7%A7%8D%E6%B5%93%E7%BC%A9%E7%89%88).md) 的最后，简单介绍了 lodash 中的数组去重方法 `_.uniq`，它可以实现我们日常工作中的去重需求，能够去重 `NaN`，并保留 `{...}`。

今天要讲的，是我从 _.uniq 的源码实现文件 baseUniq.js 中学到的几个很基础，却又容易被忽略的知识点。

&nbsp;

## 三个 API

让我们先从三个功能相近的 API 讲起，他们分别是：`_.uniq`、`_.uniqBy`、`_.uniqWith`。它们三个背后的实现文件，都指向了 **.internal** 下的 **baseUniq.js**。

区别在于 _.uniq 只需传入一个源数组 array， _.uniqBy 相较于 _.uniq 要多传一个迭代器 iteratee，而 _.uniqWith 要多传一个比较器 comparator。`iteratee` 和 `comparator` 的用法，会在后面说到。

以 _.uniqWith 为例，它是这样调用 _.baseUniq 的：
```
function uniqWith(array, comparator) {
  comparator = typeof comparator == 'function' ? comparator : undefined
  return (array != null && array.length)
    ? baseUniq(array, undefined, comparator)
    : []
}
```

&nbsp;

## baseUniq 的实现原理

baseUniq 的源码并不多，但比较绕。先贴一下的源码。
```
const LARGE_ARRAY_SIZE = 200

function baseUniq(array, iteratee, comparator) {
  let index = -1
  let includes = arrayIncludes
  let isCommon = true

  const { length } = array
  const result = []
  let seen = result

  if (comparator) {
    isCommon = false
    includes = arrayIncludesWith
  }
  else if (length >= LARGE_ARRAY_SIZE) {
    const set = iteratee ? null : createSet(array)
    if (set) {
      return setToArray(set)
    }
    isCommon = false
    includes = cacheHas
    seen = new SetCache
  }
  else {
    seen = iteratee ? [] : result
  }
  outer:
  while (++index < length) {
    let value = array[index]
    const computed = iteratee ? iteratee(value) : value

    value = (comparator || value !== 0) ? value : 0
    if (isCommon && computed === computed) {
      let seenIndex = seen.length
      while (seenIndex--) {
        if (seen[seenIndex] === computed) {
          continue outer
        }
      }
      if (iteratee) {
        seen.push(computed)
      }
      result.push(value)
    }
    else if (!includes(seen, computed, comparator)) {
      if (seen !== result) {
        seen.push(computed)
      }
      result.push(value)
    }
  }
  return result
}
```

为了兼容刚才说的三个 API，就产生了不少的干扰项。如果先从 _.uniq 入手，去掉 iteratee 和 comparator 的干扰，就会清晰不少。

```
function baseUniq(array) {
    let index = -1
    const { length } = array
    const result = []

    if (length >= 200) {
        const set = createSet(array)
        return setToArray(set)
    }

    outer:
    while (++index < length) {
        const value = array[index]
        if (value === value) {
            let resultIndex = result.length
            while (resultIndex--) {
                if (result[resultIndex] === value) {
                    continue outer
                }
            }
            result.push(value)
        } else if (!includes(seen, value)) {
            result.push(value)
        }
    }
    return result
}
```

这里有 2 个知识点。

#### 知识点一、NaN === NaN 吗？

在源码中有一个判断 `value === value`，乍一看，会觉得这是句废话！？！但其实，这是为了过滤 NaN 的情况。

MDN 中对 NaN 的解释是：它是一个全局对象的属性，初始值就是 NaN。它通常都是在计算失败时，作为 Math 的某个方法的返回值出现的。

判断一个值是否是 NaN，必须使用 **Number.isNaN()** 或 **isNaN()**，在执行自比较之中：NaN，也只有 NaN，比较之中不等于它自己。

```
NaN === NaN;        // false
Number.NaN === NaN; // false
isNaN(NaN);         // true
isNaN(Number.NaN);  // true
```

所以，在源码中，当遇到 `NaN` 的情况时，baseUniq 会转而去执行 `!includes(seen, value)` 的判断，去处理 NaN 。

#### 知识点二、冒号的特殊作用
在源码的主体部分，while 语句之前，有一行 `outer:`，它是干什么用的呢？ while 中还有一个 while 的内部，有一行 `continue outer`，从语义上理解，好像是继续执行 `outer`，这又是种什么写法呢？

```
outer:
while (++index < length) {
    ...
    while (resultIndex--) {
        if (result[resultIndex] === value) {
            continue outer
        }
    }
}
```

我们都知道 Javascript 中，常用到冒号的地方有三处，分别是：**A ? B : C 三元操作符、switch case 语句中、对象的键值对组成**。

但其实还有一种并不常见的特殊作用：`标签语句`。在 Javascript 中，任何语句都可以通过在它前面加上标志符和冒号来标记(`identifier: statement`)，这样就可以在任何地方使用该标记，最常用于循环语句中。

所以，在源码中，outer 只是看着有点不习惯，多看两遍就好了，语义上还是很好理解的。

&nbsp;

## _.uniqBy 的 iteratee

_.uniqBy 可根据指定的 key 给一个对象数组去重，一个官网的例子如下：
```
// The `_.property` iteratee shorthand.
_.uniqBy([{ 'x': 1 }, { 'x': 2 }, { 'x': 1 }], 'x');
// => [{ 'x': 1 }, { 'x': 2 }]
```

这里的 `'x'` 是 `_.property('x')` 的缩写，它指的就是 **iteratee**。

从给出的例子和语义上看，还挺好理解的。但是为什么 _.property 就能实现对象数组的去重了呢？它又是如何实现的呢？
```
@param {Array|string} path The path of the property to get.
@returns {Function} Returns the new accessor function.

function property(path) {
  return isKey(path) ? baseProperty(toKey(path)) : basePropertyDeep(path)
}
```

从注释看，property 方法会返回一个 `Function`，再看 baseProperty 的实现：

```
@param {string} key The key of the property to get.
@returns {Function} Returns the new accessor function.

function baseProperty(key) {
  return (object) => object == null ? undefined : object[key]
}
```

咦？怎么返回的还是个 `Function` ？感觉它什么也没干呀，那个参数 `object` 又是哪里来的？

#### 知识点三、纯函数的概念

纯函数，是函数式编程中的概念，它代表这样一类函数：**对于指定输出，返回指定的结果。不存在副作用**。
```
// 这是一个简单的纯函数
const addByOne = x => x + 1;
```

也就是说，纯函数的返回值只依赖其参数，函数体内不能存在任何副作用。如果是同样的参数，则一定能得到一致的返回结果。
```
function baseProperty(key) {
  return (object) => object == null ? undefined : object[key]
}
```

baseProperty 返回的就是一个纯函数，在符合条件的情况下，输出 `object[key]`。在函数式编程中，函数是“一等公民”，它可以只是根据参数，做简单的组合操作，再作为别的函数的返回值。

所以，在源码中，object 是调用 baseProperty 时传入的对象。 baseProperty 的作用，是返回期望结果为 object[key] 的函数。

&nbsp;

## _.uniqWith 的 comparator

还是先从官网的小例子说起，它会完全地给对象中所有的键值对，进行比较。
```
var objects = [{ 'x': 1, 'y': 2 }, { 'x': 2, 'y': 1 }, { 'x': 1, 'y': 2 }];

_.uniqWith(objects, _.isEqual);
// => [{ 'x': 1, 'y': 2 }, { 'x': 2, 'y': 1 }]
```

而在 baseUniq 的源码中，可以看到最终的实现，需要依赖 arrayIncludesWith 方法，以下是它的源码：
```
function arrayIncludesWith(array, target, comparator) {
  if (array == null) {
    return false
  }

  for (const value of array) {
    if (comparator(target, value)) {
      return true
    }
  }
  return false
}
```

arrayIncludesWith 没什么复杂的。comparator 作为一个参数传入，将 `target` 和 `array` 的每个 value 进行处理。从官网的例子看，_.isEqual 就是 comparator，就是要比较它们是否相等。

接着就追溯到了 _.isEqual 的源码，它的实现文件是 baseIsEqualDeep.js。在里面看到一个让我犯迷糊的写法，这是一个判断。

```
/** Used to check objects for own properties. */
const hasOwnProperty = Object.prototype.hasOwnProperty
...

const objIsWrapped = objIsObj && hasOwnProperty.call(object, '__wrapped__')
```

hasOwnProperty ？call， '__wrapped__' ？

#### 知识点四、对象的 hasOwnProperty

再次查找到了 MDN 的解释：所有继承了 Object 的对象都会继承到 hasOwnProperty 方法。它可以用来检测一个对象是否含有特定的自身属性；会忽略掉那些从原型链上继承到的属性。
```
o = new Object();
o.prop = 'exists';
o.hasOwnProperty('prop');             // 返回 true
o.hasOwnProperty('toString');         // 返回 false
o.hasOwnProperty('hasOwnProperty');   // 返回 false
```

call 的用法可以参考这篇 [细说 call、apply 以及 bind 的区别和用法](https://juejin.im/post/5c493086f265da6115111ce4)。

那么 `hasOwnProperty.call(object, '__wrapped__')` 的意思就是，判断 object 这个对象上是否存在 '__wrapped__' 这个自身属性。

__wrapped__ 是什么属性？这就要说到 lodash 的延迟计算方法 _.chain，它是一种函数式风格，从名字就可以看出，它实现的是一种链式的写法。比如下面这个例子：
```
var names = _.chain(users)
  .map(function(user){
    return user.user;
  })
  .join(" , ")
  .value();
```

如果你没有显样的调用value方法，使其立即执行的话，将会得到如下的LodashWrapper延迟表达式：

```
LodashWrapper {__wrapped__: LazyWrapper, __actions__: Array[1], __chain__: true, constructor: function, after: function…}
```

因为延迟表达式的存在，因此我们可以多次增加方法链，但这并不会被执行，所以不会存在性能的问题，最后直到我们需要使用的时候，使用 `value()` 显式立即执行即可。

所以，在 baseIsEqualDeep 源码中，才需要做 hasOwnProperty 的判断，然后在需要的情况下，执行 `object.value()`。

&nbsp;

## 总结

阅读源码，在一开始会比较困难，因为会遇到一些看不明白的写法。就像一开始我卡在了 value === value 的写法，不明白它的用意。一旦知道了是为了过滤 NaN 用的，那后面就会通畅很多了。

所以，**阅读源码，是一种很棒的重温基础知识的方式**。遇到看不明白的点，不要放过，多查多问多看，才能**不断地夯实基础，读懂更多的源码思想，体会更多的原生精髓**。如果我在一开始看到 value === value 时就放弃了，那或许就不会有今天的这篇文章了。


&nbsp;

**PS：欢迎关注我的公众号 “超哥前端小栈”，交流更多的想法与技术。**

![wechat qrCode](https://github.com/micherwa/blogs/blob/master/images/wechat_qrCode.jpg)

&nbsp;

*Posted on 2019-03-06*
