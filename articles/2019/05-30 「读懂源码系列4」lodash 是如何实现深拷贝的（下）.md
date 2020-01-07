## 前言

接着上一篇文章 [lodash 是如何实现深拷贝的（上）]()，今天会继续解读 _.cloneDeep 的源码，来看看 lodash 是如何处理对象、函数、循环引用等的深拷贝问题的。

&nbsp;

## baseClone 的源码实现

先回顾一下它的源码，以及一些关键的注释

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

#### 处理对象和函数

一些主要的判断入口，已经加上了注释。

```
const isArr = Array.isArray(value)
const tag = getTag(value)

if (isArr) {
    ... // 刚才数组的处理
} else {
    // 开始处理对象
    // 对象是函数的标志位
    const isFunc = typeof value == 'function'

    // 处理 Buffer（缓冲区）对象
    if (isBuffer(value)) {
        return cloneBuffer(value, isDeep)
    }

    // 如果 tag 是 '[object Object]'
    // 或 tag 是 '[object Arguments]'
    // 或 是函数但没有父对象（object 由 baseClone 传入，是 value 的父对象）
    if (tag == objectTag || tag == argsTag || (isFunc && !object)) {
        // 初始化 result
        // 如果是原型链或函数时，设置为空对象
        // 否则，新开一个对象，并将源对象的键值对依次拷贝进去
        result = (isFlat || isFunc) ? {} : initCloneObject(value)
        if (!isDeep) {
            // 进入对象的浅拷贝
            return isFlat
            // 如果是原型链，则需要拷贝自身，还有继承的 symbols
            ? copySymbolsIn(value, copyObject(value, keysIn(value), result))
            // 否则，只要拷贝自身的 symbols
            : copySymbols(value, Object.assign(result, value))
        }
    } else {
        // 是函数 或者 不是error类型 或者 不是weakmap类型时
        if (isFunc || !cloneableTags[tag]) {
            return object ? value : {}
        }
        // 按需要初始化 cloneableTags 对象中剩余的类型
        result = initCloneByTag(value, tag, isDeep)
    }
}
```

其中，`isBuffer` 会处理 Buffer 类的拷贝，它是 Node.js 中的概念，用来创建一个专门存放二进制数据的缓存区，可以让 Node.js 处理二进制数据。

在 baseClone 的外面，还定义了一个对象 cloneableTags，里面只有 error 和 weakmap 类型会返回 false，所以 `!cloneableTags[tag]` 的意思就是，不是 error 或 weakmap 类型。

接下来，来看如何初始化一个新的 Object 对象。

```
function initCloneObject(object) {
    return (typeof object.constructor == 'function' && !isPrototype(object))
        ? Object.create(Object.getPrototypeOf(object))
        : {}
}

// ./isPrototype.js
const objectProto = Object.prototype
// 用于检测自己是否在自己的原型链上
function isPrototype(value) {
    const Ctor = value && value.constructor
    // 如果 value 是函数，则取出该函数的原型对象
    // 否则，取出对象的原型对象
    const proto = (typeof Ctor == 'function' && Ctor.prototype) || objectProto

    return value === proto
}
```

其中，`typeof object.constructor == 'function'` 的判断，是为了确定 value 的类型是对象或数组。

然后用 `Object.create` 生成新的对象。`Object.create()` 方法用于创建一个新对象，使用现有的对象来提供新创建的对象的 `__proto__`。

`object.constructor` 相当于 `new Object()`，而 Object 的构造函数是一个函数对象。

```
const obj = new Object();

console.log(typeof obj.constructor);
// 'function'
```

对象的原型，可以通过 `Object.getPrototypeOf(obj)` 获取，它相当于过去使用的 `__proto__`。

`initCloneByTag` 方法会处理剩余的多种类型的拷贝，有原始类型，也有如 `dateTag`、`dataViewTag`、`float32Tag`、`int16Tag`、`mapTag`、`setTag`、`regexpTag` 等等。

其中，`cloneTypedArray` 方法用于拷贝类型数组。类型数组，是一种类似数组的对象，它由 ArrayBuffer、TypedArray、DataView 三类对象构成，通过这些对象为 JavaScript 提供了访问二进制数据的能力。

#### 循环引用

```
// 如果有 stack 作为参数传入，就用参数中的 stack
// 不然就 new 一个 Stack
stack || (stack = new Stack)
const stacked = stack.get(value)
if (stacked) {
    return stacked
}
stack.set(value, result)
```

与 [「前端面试题系列9」浅拷贝与深拷贝的含义、区别及实现](https://juejin.im/post/5cab479cf265da039d325df4) 最后提到的 cloneForce 方案类似，利用了栈来解决循环引用的问题。

如果 `stacked` 有值，则表明已经在栈中存在，不然就 `value` 和 `result` 入栈。在 Stack 中的 set 方法：

```
constructor(entries) {
    const data = this.__data__ = new ListCache(entries)
    this.size = data.size
}

set(key, value) {
    let data = this.__data__
    // data 是否在 ListCache 的构造函数中存在
    if (data instanceof ListCache) {
        const pairs = data.__data__
        // LARGE_ARRAY_SIZE 为 200
        if (pairs.length < LARGE_ARRAY_SIZE - 1) {
            pairs.push([key, value])
            this.size = ++data.size
            return this
        }
        // 超出200，则重置 data
        data = this.__data__ = new MapCache(pairs)
    }
    // data 不在 ListCache 的构造函数中，则直接进行 set 操作
    data.set(key, value)
    this.size = data.size
    return this
}
```

#### Map 和 Set

这两个类型的深拷贝利用了递归的思想，只是添加元素的方式有区别，`Map` 用 `set`，`Set` 用 `add`。

```
if (tag == mapTag) {
    value.forEach((subValue, key) => {
      result.set(key, baseClone(subValue, bitmask, customizer, key, value, stack))
    })
    return result
  }

  if (tag == setTag) {
    value.forEach((subValue) => {
      result.add(baseClone(subValue, bitmask, customizer, subValue, value, stack))
    })
    return result
}
```

#### Symbol 和 原型链

```
// 获取数组 keys
const keysFunc = isFull
    ? (isFlat ? getAllKeysIn : getAllKeys)
    : (isFlat ? keysIn : keys)

const props = isArr ? undefined : keysFunc(value)
arrayEach(props || value, (subValue, key) => {
    // 如果 props 有值，则替换 key 和 subValue
    if (props) {
        key = subValue
        subValue = value[key]
    }
    // 递归克隆（易受调用堆栈限制）
    assignValue(result, key, baseClone(subValue, bitmask, customizer, key, value, stack))
})

return result

// ./getAllKeysIn
// 返回一个包含 自身 和 原型链上的属性名 以及Symbol 的数组
function getAllKeysIn(object) {
    const result = []
    for (const key in object) {
        result.push(key)
    }
    if (!Array.isArray(object)) {
        result.push(...getSymbolsIn(object))
    }
    return result
}

// ./getAllKeys
// 返回一个包含 自身 和 Symbol 的数组
function getAllKeys(object) {
    const result = keys(object)
    if (!Array.isArray(object)) {
        result.push(...getSymbols(object))
    }
    return result
}

// ./keysIn
// 返回一个 自身 和 原型链上的属性名 的数组
function keysIn(object) {
    const result = []
    for (const key in object) {
        result.push(key)
    }
    return result
}

// ./keys
// 返回一个 自身属性名 的数组
function keys(object) {
    return isArrayLike(object)
        ? arrayLikeKeys(object)
        : Object.keys(Object(object))
}
```

最后来看下 `assignValue` 的实现。

```
// ./assignValue
const hasOwnProperty = Object.prototype.hasOwnProperty

function assignValue(object, key, value) {
  const objValue = object[key]

  if (!(hasOwnProperty.call(object, key) && eq(objValue, value))) {
    // value 非零或者可用
    if (value !== 0 || (1 / value) == (1 / objValue)) {
      baseAssignValue(object, key, value)
    }
  // value 未定义，并且 object 中没有 key
  } else if (value === undefined && !(key in object)) {
    baseAssignValue(object, key, value)
  }
}

// ./baseAssignValue
// 赋值的基础实现
function baseAssignValue(object, key, value) {
  if (key == '__proto__') {
    Object.defineProperty(object, key, {
      'configurable': true,
      'enumerable': true,
      'value': value,
      'writable': true
    })
  } else {
    object[key] = value
  }
}

// ./eq
// 比较两个值是否相等
function eq(value, other) {
  return value === other || (value !== value && other !== other)
}
```

最后的 eq 方法中的判断 `value !== value && other !== other`，这样的写法是为了判断 NaN。具体的可以参考这篇 [「读懂源码系列2」我从 lodash 源码中学到的几个知识点](https://github.com/micherwa/blogs/blob/master/articles/2019/03-06%20%E3%80%8C%E8%AF%BB%E6%87%82%E6%BA%90%E7%A0%81%E7%B3%BB%E5%88%972%E3%80%8D%E6%88%91%E4%BB%8E%20lodash%20%E6%BA%90%E7%A0%81%E4%B8%AD%E5%AD%A6%E5%88%B0%E7%9A%84%E5%87%A0%E4%B8%AA%E7%9F%A5%E8%AF%86%E7%82%B9.md)

&nbsp;

## 总结

cloneDeep 中囊括了各种类型的深拷贝方法，比如 node 中的 buffer，类型数组等。用了栈的思想，解决循环引用的问题。Map 和 Set 的添加元素方法比较类似，分别为 set 和 add。NaN 是不等于自身的。

深拷贝的源码解读，到此已经完结了。本篇的写作过程，同样地耗费了好几个晚上的时间，感觉真的是自己在跟自己较劲。只因为我想尽可能地把源码的实现过程说明白，其中查找资料外加理解思考，就耗费了许多时间，好在最终没有放弃，收获也是颇丰的，一些从源码中学到的技巧，也被我用到了实际项目中，提升了性能与可读性。。

近阶段因为工作原因，写文章有所懈怠了，痛定思痛还是要继续写下去。自此，《超哥前端小栈》恢复更新，同时每篇文章也会同步更新到 [掘金](https://juejin.im/user/5a5d4522518825732b19d364)、[segmentfault](https://segmentfault.com/u/micherwa) 和 [github](https://github.com/micherwa) 上。

个人的时间精力有限，在表述上有纰漏的地方，还望读者能多加指正，多多支持，期待能有更多的交流，感谢~


&nbsp;

**PS：欢迎关注我的公众号 “超哥前端小栈”，交流更多的想法与技术。**

![wechat qrCode](https://github.com/micherwa/blogs/blob/master/images/wechat_qrCode.jpg)

&nbsp;

*Posted on 2019-05-30*
