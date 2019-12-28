## 缩进与分号
- 使用soft tab（4个空格）。
- 每条语句结尾都需要有分号。
```
/* 变量申明 */
var x = 1;

/* 表达式 */
x++;

/* do-while */
do {
    x++;
} while (x < 10);
```

&nbsp;

## 空格
不需要空格的情况：
- 对象的属性名后
- 前缀一元运算符后，后缀一元运算符前
- 函数调用括号前
- 无论是函数声明还是函数表达式，'('前不要空格
- 数组的'['后和']'前，对象的'{'后和'}'前，运算符'('后和')'前
```
// 一元运算符 和 三元运算符
++x;
y++;
z = x ? 1 : 2;

// 对象的属性
var a = {
    b: 1
};

// 数组
var a = [1, 2];

// 运算符
var a = (1 + 2) * 3;
```
需要空格的情况：
- 二元运算符前后（加减乘除等）
- 三元运算符'?:'前后
- 代码块'{'前
- 下列关键字前：else, while, catch, finally
- 下列关键字后：if, else, for, while, do, switch, case, try,catch, finally, with, return, typeof
- 单行注释'//'后（若单行注释和代码同行，则'//'前也需要）多行注释'*'后对象的属性值前
- for循环，分号后留有一个空格，前置条件如果有多个，逗号后留一个空格
- 无论是函数声明还是函数表达式，'{'前一定要有空格
- 函数的参数之间
```
var doSomething = function(a, b, c) {
    // do something
};

//  '('前无空格
doSomething(item);

// 循环
for (i = 0; i < 6; i++) {
    x++;
}
```

&nbsp;

## 空行
- 变量声明后（当变量声明在代码块的最后一行时，则无需空行）
- 注释前（当注释在代码块的第一行时，则无需空行）
- 代码块后（在函数调用、数组、对象中则无需空行）
- 文件最后保留一个空行
```
// 变量申明后，需要空行
var x = 1;

// 当变量申明位于最后一行时，无需空行
if (x >= 1) {
    var y = x + 1;
}

var a = 2;

// 需要在此注释之前空行
a++;

function b() {
    // 当注释位于代码块首行时，无需空行
    return a;
}

// 代码块之后需要空行
for (var i = 0; i < 2; i++) {
    if (true) {
        return false;
    }

    continue;
}

var obj = {
    foo: function() {
        return 1;
    },

    bar: function() {
        return 2;
    }
};

```

&nbsp;

## 换行
换行的地方，行末必须有','或者运算符。

需要换行的情况：
- 代码块'{'后和'}'前
- 变量赋值后
- 判断语句中的条件过多过长

不需要换行的情况：
- 下列关键字后：else, catch, finally
- 代码块'{'前
```
var a = {
    b: 1,
    c: 2
};

if (condition) {
    ...
} else {
    ...
}

try {
    ...
} catch (e) {
    ...
} finally {
    ...
}

function test() {
    ...
}

var a,
    foo = 7,
    b, c, bar = 8;

// 判断语句中的条件过多过长
if (boolean1 &&
    boolean2 &&
    boolean3) {
    // do someting
}

```

&nbsp;

## 注释
单行注释
- 双斜线后，必须跟一个空格；
- 缩进与下一行代码保持一致；
- 不要写在代码行之后。
```
if (condition) {
    // 缩进与下一行代码保持一致
    allowed();
}
```
多行注释
- 最少三行, '*'后跟一个空格，具体参照右边的写法；
- 适合的场景：难于理解的代码段、可能存在错误的代码段、浏览器特殊的HACK代码、业务逻辑强相关的代码
```
/*
 * one space after '*'
 */
var x = 1;
```
文档注释
参考jsdoc，多用于注释函数与类。
```
/**
 * @func
 * @desc 一个带参数的函数
 * @param {string} a - 参数a
 * @param {number} b=1 - 参数b默认值为1
 * @param {string} c=1 - 参数c有两种支持的取值</br>1—表示x</br>2—表示xx
 * @param {object} d - 参数d为一个对象
 * @param {string} d.e - 参数d的e属性
 * @param {string} d.f - 参数d的f属性
 * @param {object[]} g - 参数g为一个对象数组
 * @param {string} g.h - 参数g数组中一项的h属性
 * @param {string} g.i - 参数g数组中一项的i属性
 * @param {string} [j] - 参数j是一个可选参数
 */
function foo(a, b, c, d, g, j) {
    ...
}
```

&nbsp;

## 引号
最外层统一使用单引号。
```
let y = 'foo';
let z = '<div id="test"></div>';
```

&nbsp;

## 变量命名
- 标准变量采用驼峰式命名（除了对象的属性外）
- 'ID'在变量名中全大写
- 'Android'在变量名中大写第一个字母
- 'iOS'在变量名中小写第一个，大写后两个字母
- 常量全大写，用下划线连接
- 构造函数，大写第一个字母
```
var thisIsMyName;

var goodID;

var AndroidVersion;

var iOSVersion;

var MAX_COUNT = 10;

function Person(name) {
    this.name = name;
}
```

&nbsp;

## 函数
- 无论是函数声明还是函数表达式，'('前不要空格，但'{'前一定要有空格；
- 函数调用括号前不需要空格；
- 立即执行函数外必须包一层括号；
- 不要给inline function命名；
- 参数之间用', '分隔，注意逗号后有一个空格；
- 一个函数内，最多3层缩进，专注于实现一件事。
```
var doSomething = function(item) {
    // do something
};

function doSomething(item) {
    // do something
}

doSomething(item);

// 立即执行函数
(function() {
    return 1;
})();

// inline function
var a = [1, 2, function() {
    ...
}];

// 参数间以', '分割
var doSomething = function(a, b, c) {
    // do something
};
```

&nbsp;

## 括号
下列关键字后必须有大括号（即使代码块的内容只有一行）：if, else,for, while, do, switch, try, catch, finally, with。
```
// not good
if (condition)
    doSomething();

// good
if (condition) {
    doSomething();
}
```

&nbsp;

## null
适用场景：
- 初始化一个将来可能被赋值为对象的变量
- 与已经初始化的变量做比较
- 作为一个参数为对象的函数的调用传参
- 作为一个返回对象的函数的返回值

不适用场景：
- 不要用null来判断函数调用时有无传参
- 不要与未初始化的变量做比较
```
// not good
function test(a, b) {
    if (b === null) {
        // not mean b is not supply
        ...
    }
}

// good
var a = null;

if (a === null) {
    ...
}
```

&nbsp;

## undefined
永远不要直接使用undefined进行变量判断；

使用typeof和字符串'undefined'对变量进行判断。
```
// not good
if (person === undefined) {
    ...
}

// good
if (typeof person === 'undefined') {
    ...
}
```

&nbsp;

## jshint
- 用'===', '!=='代替'==', '!='；
- for-in里一定要有hasOwnProperty的判断；
- 不要在内置对象的原型上添加方法，如Array, Date；
- 不要在内层作用域的代码里声明了变量，之后却访问到了外层作用域的同名变量；
- 变量不要先使用后声明；
- 不要在一句代码中单单使用构造函数，记得将其赋值给某个变量；
- 不要在同个作用域下声明同名变量；
- 不要在一些不需要的地方加括号，例：delete(a.b)；
- 不要使用未声明的变量（全局变量需要加到.jshintrc文件的globals属性里面）；
- 不要声明了变量却不使用；
- debugger不要出现在提交的代码里；
- 数组中不要存在空元素；
- 不要在循环内部声明函数；
- 不要像这样使用构造函数，例：new function () { ... }, new Object
```
// not good
if (a == 1) {
    a++;
}

// good
if (a === 1) {
    a++;
}

// good
for (key in obj) {
    if (obj.hasOwnProperty(key)) {
        // be sure that obj[key] belongs to the object and was not inherited
        console.log(obj[key]);
    }
}

// not good
Array.prototype.count = function(value) {
    return 4;
};

// not good
var x = 1;

function test() {
    if (true) {
        var x = 0;
    }

    x += 1;
}

// not good
function test() {
    console.log(x);

    var x = 1;
}

// not good
new Person();

// good
var person = new Person();

// not good
delete(obj.attr);

// good
delete obj.attr;

// not good
if (a = 10) {
    a++;
}

// not good
var a = [1, , , 2, 3];

// not good
var nums = [];

for (var i = 0; i < 10; i++) {
    (function(i) {
        nums[i] = function(j) {
            return i + j;
        };
    }(i));
}

// not good
var singleton = new function() {
    var privateVar;

    this.publicMethod = function() {
        privateVar = 1;
    };

    this.publicMethod2 = function() {
        privateVar = 2;
    };
};
```

&nbsp;

## 杂项
- 不要混用tab和space，或在一处使用多个tab或space；
- 对上下文this的引用只能使用'_this', 'that', 'self'其中一个来命名；
- 行尾不要有空白字符；
- switch的falling through和no default的情况一定要有注释特别说明；
- 不允许有空的代码块。
```
function Person() {
    // not good
    var me = this;

    // good
    var _this = this;

    // good
    var that = this;

    // good
    var self = this;
}
```

&nbsp;

*Posted on 2018-05-26*
