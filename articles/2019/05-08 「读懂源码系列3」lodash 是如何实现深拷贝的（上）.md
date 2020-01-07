## 前言

上一篇文章 [「前端面试题系列9」浅拷贝与深拷贝的含义、区别及实现](https://github.com/micherwa/blogs/blob/master/articles/2019/04-11%20%E3%80%8C%E5%89%8D%E7%AB%AF%E9%9D%A2%E8%AF%95%E9%A2%98%E7%B3%BB%E5%88%979%E3%80%8D%E6%B5%85%E6%8B%B7%E8%B4%9D%E4%B8%8E%E6%B7%B1%E6%8B%B7%E8%B4%9D%E7%9A%84%E5%90%AB%E4%B9%89%E3%80%81%E5%8C%BA%E5%88%AB%E5%8F%8A%E5%AE%9E%E7%8E%B0.md) 中提到了深拷贝的实现方法，从递归调用，到 JSON，再到终极方案 cloneForce。

不经让我想到，lodash 中的 `_.cloneDeep` 方法。它是如何实现深拷贝的呢？今天，就让我们来具体地解读一下 _.cloneDeep 的源码实现。

源码中的内容比较多，为了能将知识点讲明白，也为了更好的阅读体验，将会分为上下 2 篇进行解读。今天主要会涉及位掩码、对象判断、数组和正则的深拷贝写法。

ok，现在就让我们深入源码，共同探索吧~

&nbsp;

## _.cloneDeep 的源码实现

它的源码内容很少，因为主要还是靠 baseClone 去实现。

```
/** Used to compose bitmasks for cloning. */
const CLONE_DEEP_FLAG = 1
const CLONE_SYMBOLS_FLAG = 4

function cloneDeep(value) {
  return baseClone(value, CLONE_DEEP_FLAG | CLONE_SYMBOLS_FLAG)
}
```

刚看到前两行的常量就懵了，它们的用意是什么？然后，传入 baseClone 的第二个参数，似乎还将那两个常量做了运算，其结果是什么？这么做的目的是什么？

一番查找之后，终于明白这里其实涉及到了 `位掩码` 与 `位运算` 的概念。下面就来详细讲解一下。

#### 位掩码技术

回到第一行注释：`Used to compose bitmasks for cloning`。意思是，用于构成克隆方法的位掩码。

从注释看，这里的 `CLONE_DEEP_FLAG` 和 `CLONE_SYMBOLS_FLAG` 就是位掩码了，而 `CLONE_DEEP_FLAG | CLONE_SYMBOLS_FLAG` 其实是 **位运算** 中的 `按位或` 方法。

这里有个不常见的概念：`位运算`。MDN 上对位运算的解释是：它经常被用来创建、处理以及读取标志位序列——一种类似二进制的变量。虽然可以使用变量代替标志位序列，但是这样可以节省内存（1/32）。

不过实际开发中，位运算用得很少，主要是因为位运算操作的是二进制位，对开发者来说不太好理解。用得少，就容易生疏。但实际上，位运算是一种很棒的思想，它计算得更快，代码量还更少。位运算，常用于处理同时存在多个布尔选项的情形。掩码中的每个选项的值都是 2 的幂，位运算是 32 位的。

在计算机程序的世界里，所有的数据都是以二进制的形式储存的。位运算，说白了就是直接对某个数据在内存中的二进制位，进行运算操作。比如 `&`、`|`、`~`、`^`、`>>`，这些都是 按位运算符，它们有一些神奇的用法。以系统权限为例：
```
const PERMISSION_A = 1; // 0001
const PERMISSION_B = 2; // 0010
const PERMISSION_C = 4; // 0100
const PERMISSION_D = 8; // 1000

// 当一个用户同时拥有 权限A 和 权限C 时，就产生了一个新的权限
const mask = PERMISSION_A | PERMISSION_C; // 0101，十进制为 5

// 判断该用户是否有 权限C，可以取出 权限C 的位掩码
if (mask & PERMISSION_C) {
    ...
}

// 该用户没有 权限A，也没有 权限C
const mask2 = ~(PERMISSION_A | PERMISSION_C); // ~0101 => 1010

// 取出 与权限A 不同的部分
const mask3 = mask ^ PERMISSION_A; // 0101 ^ 0001 => 0100
```

回到源码的 `CLONE_DEEP_FLAG | CLONE_SYMBOLS_FLAG` 就得到一个新的结果传入 baseClone 中，十进制为 5，至于它是用来干什么的，就需要继续深入到 baseClone 的源码中去看了。

&nbsp;

## baseClone 的源码实现

先贴一下源码，其中一些关键的判断已经做了注释
```
function baseClone(value, bitmask, customizer, key, object, stack) {
  let result
  // 根据位掩码，切分判断入口
  const isDeep = bitmask & CLONE_DEEP_FLAG
  const isFlat = bitmask & CLONE_FLAT_FLAG
  const isFull = bitmask & CLONE_SYMBOLS_FLAG

  // 自定义 clone 方法，用于 _.cloneWith
  if (customizer) {
    result = object ? customizer(value, key, object, stack) : customizer(value)
  }
  if (result !== undefined) {
    return result
  }

  // 过滤出原始类型，直接返回
  if (!isObject(value)) {
    return value
  }

  const isArr = Array.isArray(value)
  const tag = getTag(value)
  if (isArr) {
    // 处理数组
    result = initCloneArray(value)
    if (!isDeep) {
      // 浅拷贝数组
      return copyArray(value, result)
    }
  } else {
    // 处理对象
    const isFunc = typeof value == 'function'

    if (isBuffer(value)) {
      return cloneBuffer(value, isDeep)
    }
    if (tag == objectTag || tag == argsTag || (isFunc && !object)) {
      result = (isFlat || isFunc) ? {} : initCloneObject(value)
      if (!isDeep) {
        return isFlat
          ? copySymbolsIn(value, copyObject(value, keysIn(value), result))
          : copySymbols(value, Object.assign(result, value))
      }
    } else {
      if (isFunc || !cloneableTags[tag]) {
        return object ? value : {}
      }
      result = initCloneByTag(value, tag, isDeep)
    }
  }
  // 用 “栈” 处理循环引用
  stack || (stack = new Stack)
  const stacked = stack.get(value)
  if (stacked) {
    return stacked
  }
  stack.set(value, result)

  // 处理 Map
  if (tag == mapTag) {
    value.forEach((subValue, key) => {
      result.set(key, baseClone(subValue, bitmask, customizer, key, value, stack))
    })
    return result
  }

  // 处理 Set
  if (tag == setTag) {
    value.forEach((subValue) => {
      result.add(baseClone(subValue, bitmask, customizer, subValue, value, stack))
    })
    return result
  }

  // 处理 typedArray
  if (isTypedArray(value)) {
    return result
  }

  const keysFunc = isFull
    ? (isFlat ? getAllKeysIn : getAllKeys)
    : (isFlat ? keysIn : keys)

  const props = isArr ? undefined : keysFunc(value)

  // 遍历赋值
  arrayEach(props || value, (subValue, key) => {
    if (props) {
      key = subValue
      subValue = value[key]
    }
    // Recursively populate clone (susceptible to call stack limits).
    assignValue(result, key, baseClone(subValue, bitmask, customizer, key, value, stack))
  })

  return result
}
```

#### 位掩码的作用

```
/** Used to compose bitmasks for cloning. */
const CLONE_DEEP_FLAG = 1 // 深拷贝标志位
const CLONE_FLAT_FLAG = 2 // 原型链标志位
const CLONE_SYMBOLS_FLAG = 4 // Symbol 标志位

function baseClone(value, bitmask, customizer, key, object, stack) {
    // 根据位掩码，取出位掩码，切分判断入口，bitmask 的十进制为 5
    const isDeep = bitmask & CLONE_DEEP_FLAG // 5 & 1 => 1 => true
    const isFlat = bitmask & CLONE_FLAT_FLAG // 5 & 2 => 0 => false
    const isFull = bitmask & CLONE_SYMBOLS_FLAG // 5 & 4 => 4 => true
    ...
}
```

每个常量基本都加了注释，之前传入 baseClone 的 bitmask 为十进制的 5，其目的就是为了在 baseClone 中进行判断入口的切分。

#### 是否为对象的判断

```
// 如果不是对象，则直接返回该值
if (!isObject(value)) {
    return value
}

// ./isObject.js
function isObject(value) {
  const type = typeof value
  return value != null && (type == 'object' || type == 'function')
}
```

这里需要说的就是，是否为对象的判断。用的基本方法是 `typeof`，但是因为 typeof null 的值也是 'object'，所以最后的 return 需要对 null 做额外处理。

#### 处理数组和正则

```
const isArr = Array.isArray(value)

if (isArr) {
    result = initCloneArray(value)
    if (!isDeep) {
        return copyArray(value, result)
    }
} else {
    ... // 非数组的处理
}

// 用于检测对象自身的属性
const hasOwnProperty = Object.prototype.hasOwnProperty

// 初始化需要克隆的数组
function initCloneArray(array) {
    const { length } = array
    const result = new array.constructor(length)

    // Add properties assigned by `RegExp#exec`.
    if (length && typeof array[0] == 'string' && hasOwnProperty.call(array, 'index')) {
        result.index = array.index
        result.input = array.input
    }
    return result
}
```

为了不干扰源数组的数据，这里首先会用 initCloneArray 初始化一个全新的数组。

其中，`new array.constructor(length)` 相当于 `new Array(length)`，只是换了种不常见的写法，作用是一样的。

接下来的这个判断，让我一头雾水。
```
// Add properties assigned by `RegExp#exec`.
if (length && typeof array[0] == 'string' && hasOwnProperty.call(array, 'index')) {
    result.index = array.index
    result.input = array.input
}
```

判断条件首先确定 length > 0，然后 array[0] 的类型是 string，最后 array 拥有 index 这个属性。

看到判断条件里的两条执行语句更懵了，需要赋值 `index` 和 `input`，这又是为什么？/(ㄒoㄒ)/~~

回头看到第一行注释，有个关键点 `RegExp#exec`。[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/RegExp/exec) 中给的解释：exec() 方法在一个指定字符串中执行一个搜索匹配。返回一个结果数组或 null。文档下方有个例子：
```
var re = /quick\s(brown).+?(jumps)/ig;
var result = re.exec('The Quick Brown Fox Jumps Over The Lazy Dog');
console.log(result);

// 输出的 result 是一个数组，有 3 个元素和 4 个属性
// 0: "Quick Brown Fox Jumps"
// 1: "Brown"
// 2: "Jumps"
// groups: undefined
// index: 4
// input: "The Quick Brown Fox Jumps Over The Lazy Dog"
// length: 3
```

哇哦~ 原来 `index` 和 `input` 在这里。所以，源码中的为何要那样赋值，就迎刃而解了。

再回到 baseClone 中来，如果不是深拷贝，那就只要做数组的第一层数据的赋值即可。

```
if (!isDeep) {
    return copyArray(value, result)
}

// ./copyArray.js
function copyArray(source, array) {
  let index = -1
  const length = source.length

  array || (array = new Array(length))
  while (++index < length) {
    array[index] = source[index]
  }
  return array
}
```

&nbsp;

## 总结

位掩码技术，是一种很棒的思想，可以写出更为简洁的代码，运行得也更快。对象的判断，需要特别注意 null，它的 typeof 值 也是 object。正则的 exec() 方法会返回一个结果数组或 null，其中就会有 index 和 input 属性。

阅读源码的过程比较痛苦，深感自身的不足。从不懂到查阅资料，再到写出来，耗费了我大量的时间，不过写作的过程也给了我不小的收获。修行之路任重而道远，给自己打打气，继续砥砺前行吧。


&nbsp;

**PS：欢迎关注我的公众号 “超哥前端小栈”，交流更多的想法与技术。**

![wechat qrCode](https://github.com/micherwa/blogs/blob/master/images/wechat_qrCode.jpg)

&nbsp;

*Posted on 2019-05-08*