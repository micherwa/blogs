![bg-20190313-01.jpg](https://github.com/micherwa/blogs/blob/master/images/2019/bg-20190313-01.jpg)

## 前言

上一篇文章[「前端面试题系列8」数组去重(10 种浓缩版) ](https://github.com/micherwa/blogs/blob/master/articles/2019/02-28%20%E3%80%8C%E5%89%8D%E7%AB%AF%E9%9D%A2%E8%AF%95%E9%A2%98%E7%B3%BB%E5%88%978%E3%80%8D%E6%95%B0%E7%BB%84%E5%8E%BB%E9%87%8D(10%20%E7%A7%8D%E6%B5%93%E7%BC%A9%E7%89%88).md)中提到了不少数组的常用操作。

今天，会更具体地将数组的常用操作进行归纳和汇总，以便备不时之需。每组方法都会配以示例说明，有时我也会忘了某个方法是否会返回一个新的数组，如果你也有类似的困惑，那么看这篇就够了。希望能帮到有需要的同学。

&nbsp;

## ES5 中 Array 操作

#### 1、forEach

Array 方法中最基本的一个，就是遍历，循环。基本用法：`[].forEach(function(item, index, array) {});`
```
const array = [1, 2, 3];
const result = [];

array.forEach(function(item) {
    result.push(item);
});
console.log(result); // [1, 2, 3]
```

#### 2、map

map 方法的作用不难理解，“映射”嘛，也就是原数组被 “映射” 成对应的新数组。基本用法跟 forEach 方法类似：`[].map(function(item, index, array) {});` 下面这个例子是数值项求平方：
```
const data = [1, 2, 3, 4];
const arrayOfSquares = data.map(function (item) {
    return item * item;
});
alert(arrayOfSquares); // 1, 4, 9, 16
```

#### 3、filter

filter 为 “过滤”、“筛选” 之意。指数组 filter 后，返回过滤后的新数组。用法跟 map 极为相似：`[].filter(function(item, index, array) {});`

filter 的 callback 函数，需要返回布尔值 true 或 false。返回值只要 **弱等于** Boolean 就行，看下面这个例子：
```
const data = [0, 1, 2, 3];
const arrayFilter = data.filter(function(item) {
    return item;
});
console.log(arrayFilter); // [1, 2, 3]
```

#### 4、some 和 every

some 意指“某些”，指是否 “某些项” 合乎条件。也就是 **只要有1值个让 callback 返回 true**  就行了。基本用法：`[].som(function(item, index, array) {});`

```
const scores = [5, 8, 3, 10];
const current = 7;

function higherThanCurrent(score) {
    return score > current;
}

if (scores.some(higherThanCurrent)) {
    alert("one more");
}
```

every 跟 some 是一对好基友，同样是返回 Boolean 值。但必须满足每 1 个值都要让 callback 返回 true 才行。改动一下上面 some 的例子：

```
if (scores.every(higherThanCurrent)) {
    console.log("every is ok!");
} else {
    console.log("oh no!");
}
```

#### 5、concat

concat() 方法用于连接两个或多个数组。该方法不会改变现有的数组，仅会返回被连接数组的一个副本。

```
const arr1 = [1,2,3];
const arr2 = [4,5];
const arr3 = arr1.concat(arr2);
console.log(arr1); // [1, 2, 3]
console.log(arr3); // [1, 2, 3, 4, 5]
```

#### 6、indexOf 和 lastIndexOf

indexOf 方法在字符串中自古就有，string.indexOf(searchString, position)。数组这里的 indexOf 方法与之类似。
返回整数索引值，如果没有匹配（严格匹配），返回-1.

fromIndex可选，表示从这个位置开始搜索，若缺省或格式不合要求，使用默认值0。

```
const data = [2, 5, 7, 3, 5];
console.log(data.indexOf(5, "x"));  // 1 ("x"被忽略)
console.log(data.indexOf(5, "3")); // 4 (从3号位开始搜索)

console.log(data.indexOf(4));    // -1 (未找到)
console.log(data.indexOf("5")); // -1 (未找到，因为5 !== "5")
```

lastIndexOf 则是从后往前找。
```
const numbers = [2, 5, 9, 2];
numbers.lastIndexOf(2);     // 3
numbers.lastIndexOf(7);     // -1
numbers.lastIndexOf(2, 3);  // 3
numbers.lastIndexOf(2, 2);  // 0
numbers.lastIndexOf(2, -2); // 0
numbers.lastIndexOf(2, -1); // 3
```

#### 7、reduce 和 reduceRight

reduce 是JavaScript 1.8 中才引入的，中文意思为“归约”。基本用法：`reduce(callback[, initialValue])`

callback 函数接受4个参数：之前值(previousValue)、当前值(currentValue)、索引值(currentIndex)以及数组本身(array)。

可选的初始值(initialValue)，作为第一次调用回调函数时传给previousValue的值。也就是，为累加等操作传入起始值（额外的加值）。

```
var sum = [1, 2, 3, 4].reduce(function (previous, current, index, array) {
    return previous + current;
});
console.log(sum); // 10
```

解析：
- 因为initialValue不存在，因此一开始的previous值等于数组的第一个元素
- 从而 current 值在第一次调用的时候就是2
- 最后两个参数为索引值 index 以及数组本身 array

以下为循环执行过程：
```
// 初始设置
previous = initialValue = 1, current = 2

// 第一次迭代
previous = (1 + 2) =  3, current = 3

// 第二次迭代
previous = (3 + 3) =  6, current = 4

// 第三次迭代
previous = (6 + 4) =  10, current = undefined (退出)
```

有了reduce，我们可以轻松实现二维数组的扁平化：
```
var matrix = [
  [1, 2],
  [3, 4],
  [5, 6]
];

// 二维数组扁平化
var flatten = matrix.reduce(function (previous, current) {
    return previous.concat(current);
});
console.log(flatten); // [1, 2, 3, 4, 5, 6]
```

reduceRight 跟 reduce 相比，用法类似。实现上差异在于reduceRight是从数组的末尾开始实现。

```
const data = [1, 2, 3, 4];
const specialDiff = data.reduceRight(function (previous, current, index) {
    if (index == 0) {
        return previous + current;
    }
    return previous - current;
});
console.log(specialDiff);  // 0
```

#### 8、push 和 pop

push() 方法可向数组的末尾添加一个或多个元素，返回的是新的数组长度，会改变原数组。

```
const a = [2,3,4];
const b = a.push(5);
console.log(a);  // [2,3,4,5]
console.log(b);  // 4
// push方法可以一次添加多个元素push(data1, data2....)
```

pop() 方法用于删除并返回数组的最后一个元素。返回的是最后一个元素，会改变原数组。

```
const arr = [2,3,4];
console.log(arr.pop()); // 4
console.log(arr);  // [2,3]
```

#### 9、shift 和 unshift

shift() 方法用于把数组的第一个元素从其中删除，并返回第一个元素的值。返回第一个元素，改变原数组。

```
const arr = [2,3,4];
console.log(arr.shift()); // 2
console.log(arr);  // [3,4]
```
unshift() 方法可向数组的开头添加一个或更多元素，并返回新的长度。返回新长度，改变原数组。

```
const arr = [2,3,4,5];
console.log(arr.unshift(3,6)); // 6
console.log(arr); // [3, 6, 2, 3, 4, 5]
// tip:该方法可以不传参数,不传参数就是不增加元素。
```

#### 10、slice 和 splice

slice() 返回一个新的数组，包含从 start 到 end （不包括该元素）的 arrayObject 中的元素。返回选定的元素，该方法不会修改原数组。

```
const arr = [2,3,4,5];
console.log(arr.slice(1,3));  // [3,4]
console.log(arr);  // [2,3,4,5]
```

splice() 可删除从 index 处开始的零个或多个元素，并且用参数列表中声明的一个或多个值来替换那些被删除的元素。如果从 arrayObject 中删除了元素，则返回的是含有被删除的元素的数组。splice() 方法会直接对数组进行修改。
```
const a = [5,6,7,8];
console.log(a.splice(1,0,9)); //[]
console.log(a);  // [5, 9, 6, 7, 8]

const b = [5,6,7,8];
console.log(b.splice(1,2,3));  //v[6, 7]
console.log(b); // [5, 3, 8]
```

#### 11、sort 和 reverse

sort() 按照 Unicode code 位置排序，默认升序。

```
const fruit = ['cherries', 'apples', 'bananas'];
fruit.sort(); // ['apples', 'bananas', 'cherries']

const scores = [1, 10, 21, 2];
scores.sort(); // [1, 10, 2, 21]
```

reverse() 方法用于颠倒数组中元素的顺序。返回的是颠倒后的数组，会改变原数组。

```
const arr = [2,3,4];
console.log(arr.reverse()); // [4, 3, 2]
console.log(arr);  // [4, 3, 2]
```

#### 12、join

join() 方法用于把数组中的所有元素放入一个字符串。元素是通过指定的分隔符进行分隔的，默认使用','号分割，不改变原数组。

```
const arr = [2,3,4];
console.log(arr.join());  // 2,3,4
console.log(arr);  // [2, 3, 4]
```

#### 13、isArray

Array.isArray() 用于确定传递的值是否是一个 Array。一个比较冷门的知识点：其实 Array.prototype 也是一个数组。
```
Array.isArray([]); // true
Array.isArray(Array.prototype); // true

Array.isArray(null); // false
Array.isArray(undefined); // false
Array.isArray(18); // false
Array.isArray('Array'); // false
Array.isArray(true); // false
Array.isArray({ __proto__: Array.prototype });
```

在公用库中，一般会这么做 isArray 的判断：
```
Object.prototype.toString.call(arg) === '[object Array]';
```

&nbsp;

## ES6 新增的 Array 操作

#### 1、find 和 findIndex

find() 传入一个回调函数，找到数组中符合当前搜索规则的第一个元素，返回这个元素，并且终止搜索。
```
const arr = [1, "2", 3, 3, "2"]
console.log(arr.find(n => typeof n === "number")) // 1
```

findIndex() 与 find() 类似，只是返回的是，找到的这个元素的下标。

```
const arr = [1, "2", 3, 3, "2"]
console.log(arr.findIndex(n => typeof n === "number")) // 0
```

#### 2、fill

用指定的元素填充数组，其实就是用默认内容初始化数组。基本用法：`[].fill(value, start, end)`

该函数有三个参数：填充值(value)，填充起始位置(start，可以省略)，填充结束位置(end，可以省略，实际结束位置是end-1)。

```
// 采用一个默认值，填充数组
const arr1 = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11];
arr1.fill(7);
console.log(arr1); // [7,7,7,7,7,7,7,7,7,7,7]

// 制定开始和结束位置填充，实际填充结束位置是前一位。
const arr2 = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11];
arr2.fill(7, 2, 5);
console.log(arr2); // [1,2,7,7,7,6,7,8,9,10,11]

// 结束位置省略，从起始位置到最后。
const arr3 = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11];
arr3.fill(7, 2);
console.log(arr3); // [1,2,7,7,7,7,7,7,7,7,7]
```

#### 3、from

将类似数组的对象（array-like object）和可遍历（iterable）的对象转为真正的数组。
```
const set = new Set(1, 2, 3, 3, 4);
Array.from(set)  // [1,2,3,4]

Array.from('foo'); // ["f", "o", "o"]
```

#### 4、of

Array.of() 方法创建一个具有可变数量参数的新数组实例，而不考虑参数的数量或类型。

Array.of() 和 Array 构造函数之间的区别在于处理整数参数：Array.of(7) 创建一个具有单个元素 7 的数组，而 Array(7) 创建一个长度为7的空数组（注意：这是指一个有7个空位的数组，而不是由7个undefined组成的数组）。
```
Array.of(7);       // [7]
Array.of(1, 2, 3); // [1, 2, 3]

Array(7);          // [ , , , , , , ]
Array(1, 2, 3);    // [1, 2, 3]
```

#### 5、copyWithin

选择数组的某个下标，从该位置开始复制数组元素，默认从0开始复制。也可以指定要复制的元素范围。基本用法：`[].copyWithin(target, start, end)`

```
const arr = [1, 2, 3, 4, 5];
console.log(arr.copyWithin(3));
 // [1,2,3,1,2] 从下标为3的元素开始，复制数组，所以4, 5被替换成1, 2

const arr1 = [1, 2, 3, 4, 5];
console.log(arr1.copyWithin(3, 1));
// [1,2,3,2,3] 从下标为3的元素开始，复制数组，指定复制的第一个元素下标为1，所以4, 5被替换成2, 3

const arr2 = [1, 2, 3, 4, 5];
console.log(arr2.copyWithin(3, 1, 2));
// [1,2,3,2,5] 从下标为3的元素开始，复制数组，指定复制的第一个元素下标为1，结束位置为2，所以4被替换成2
```

#### 6、includes

判断数组中是否存在该元素，参数：查找的值、起始位置，可以替换 ES5 时代的 indexOf 判断方式。

```
const arr = [1, 2, 3];
arr.includes(2); // true
arr.includes(4); // false
```

另外，它还可以用于优化 || 的判断写法。

```
if (method === 'post' || method === 'put' || method === 'delete') {
    ...
}

// 用 includes 优化 `||` 的写法
if (['post', 'put', 'delete'].includes(method)) {
    ...
}
```

#### 7、entries、values 和 keys

entries() 返回迭代器：返回键值对

```
//数组
const arr = ['a', 'b', 'c'];
for(let v of arr.entries()) {
    console.log(v)
}
// [0, 'a'] [1, 'b'] [2, 'c']

//Set
const arr = new Set(['a', 'b', 'c']);
for(let v of arr.entries()) {
    console.log(v)
}
// ['a', 'a'] ['b', 'b'] ['c', 'c']

//Map
const arr = new Map();
arr.set('a', 'a');
arr.set('b', 'b');
for(let v of arr.entries()) {
    console.log(v)
}
// ['a', 'a'] ['b', 'b']
```

values() 返回迭代器：返回键值对的 value

```
//数组
const arr = ['a', 'b', 'c'];
for(let v of arr.values()) {
    console.log(v)
}
//'a' 'b' 'c'

//Set
const arr = new Set(['a', 'b', 'c']);
for(let v of arr.values()) {
    console.log(v)
}
// 'a' 'b' 'c'

//Map
const arr = new Map();
arr.set('a', 'a');
arr.set('b', 'b');
for(let v of arr.values()) {
    console.log(v)
}
// 'a' 'b'
```

keys() 返回迭代器：返回键值对的 key
```
//数组
const arr = ['a', 'b', 'c'];
for(let v of arr.keys()) {
    console.log(v)
}
// 0 1 2

//Set
const arr = new Set(['a', 'b', 'c']);
for(let v of arr.keys()) {
    console.log(v)
}
// 'a' 'b' 'c'

//Map
const arr = new Map();
arr.set('a', 'a');
arr.set('b', 'b');
for(let v of arr.keys()) {
    console.log(v)
}
// 'a' 'b'
```

&nbsp;

## 总结

操作 Array 的数据结构，在日常工作中会经常遇到。ES6 在 Array 的操作上，也提供了更为简便实用的方法，比如 includes、find、from 等等。

过去，我会习惯于用 lodash 的 _.find 方法，现在就可以选择拥抱原生了。


&nbsp;

**PS：欢迎关注我的公众号 “超哥前端小栈”，交流更多的想法与技术。**

![wechat qrCode](https://github.com/micherwa/blogs/blob/master/images/wechat_qrCode.jpg)

&nbsp;

*Posted on 2019-03-13*
