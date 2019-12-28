## 缩进与分号
- 使用soft tab（4个空格）。
- 每个属性声明末尾都要加分号。
```
.element {
    width: 20px;
    height: 20px;

    background-color: red;
}
```

&nbsp;

## 空格
1.不需要空格的情况：
- 属性名后
- 多个规则的分隔符','前
- `!important` '!'后
- 行末不要有多余的空格
```
/* not good */
.element {
    color :red! important;
}

/* good */
.element {
    color: red !important;
}

/* not good */
.element ,
.dialog{
    ...
}

/* good */
.element,
.dialog {
    ...
}
```

2.需要空格的情况
- 属性值前
- 选择器'>', '+', '~'前后
- '{'前
- `!important` '!'前
- `@else` 前后
- 属性值中的','后
- 注释'/*'后和'*/'前

&nbsp;

## 空行
- 文件最后保留一个空行
- '}'后最好跟一个空行，包括scss中嵌套的规则
- 属性之间需要适当的空行
```
/* not good */
.element {
    ...
}
.dialog {
    color: red;
    &:after {
        ...
    }
}

/* good */
.element {
    ...
}

.dialog {
    color: red;

    &:after {
        ...
    }
}
```

&nbsp;

## 换行
- '{'后和'}'前需要换行
- 每个属性独占一行
- 多个规则的分隔符','后
```
/* not good */
.element
{color: red; background-color: black;}

/* good */
.element {
    color: red;
    background-color: black;
}

/* not good */
.element, .dialog {
    ...
}

/* good */
.element,
.dialog {
    ...
}
```

&nbsp;

## 注释
- 统一用'/**/'（scss中也不要用'//'）；
- 缩进与下一行代码保持一致；
- 写在需要注释的代码的上方，不要跟在代码后面。
```
/* Modal header */
.modal-header {
    ...
}

/*
 * Modal header
 */
.modal-header {
    ...
}

.modal-header {
    /* 50px */
    width: 50px;
}
```

&nbsp;

## 引号
- url的内容要用引号；
- 属性选择器中的属性值需要引号。
```
.element:after {
    content: "";
    background-image: url("logo.png");
}

input[type="checkbox"] {
    ...
}
```

&nbsp;

## 命名
- 类名使用小写字母，以中划线分隔；
- id采用驼峰式命名；
- scss中的变量、函数、混合、placeholder采用驼峰式命名。
```
/* class */
.element-content {
    ...
}

/* id */
#myDialog {
    ...
}

/* 变量 */
$colorBlack: #000;

/* 函数 */
@function pxToRem($px) {
    ...
}

/* 混合 */
@mixin centerBlock {
    ...
}

/* placeholder */
%myDialog {
    ...
}
```

&nbsp;

## 颜色
颜色16进制，用小写字母表示，尽量用简写。
```
/* not good */
.element {
    color: #ABCDEF;
    background-color: #001122;
}

/* good */
.element {
    color: #abcdef;
    background-color: #012;
}
```

&nbsp;

## 媒体查询
尽量将媒体查询的规则靠近与他们相关的规则；
不要将他们一起放到一个独立的样式文件中，或者丢在文档的最底部，
这样做只会让大家以后更容易忘记他们。
```
.element {
    ...
}

@media (min-width: 480px) {
    .element {
        ...
    }
}
```

&nbsp;

## SCSS相关
1.提交的代码中不要有 @debug；
2.声明顺序：
- `@extend`
- 不包含 `@content` 的 `@include`
- 包含 `@content` 的 `@include`
- 自身属性
- 嵌套规则
3.`@import` 引入的文件不需要开头的 ' _ ' 和结尾的 '.scss'；
4.嵌套最多不能超过5层；
5.`@extend` 中使用placeholder选择器；
6.去掉不必要的父级引用符号 ` & ` 。

```
/* not good */
@import "_dialog.scss";

/* good */
@import "dialog";

/* not good */
.fatal {
    @extend .error;
}

/* good */
.fatal {
    @extend %error;
}

/* not good */
.element {
    & > .dialog {
        ...
    }
}

/* good */
.element {
    > .dialog {
        ...
    }
}
```

&nbsp;

## 一些小建议
- 不允许有空的规则；
- 元素选择器用小写字母；
- 去掉小数点前面的0；
- 去掉数字中不必要的小数点和末尾的0；
- 属性值'0'后面不要加单位；
- 同个属性不同前缀的写法需要在垂直方向保持对齐；
- 无前缀的标准属性应该写在有前缀的属性后面；
- 不要在同个规则里出现重复的属性，如果重复的属性是连续的则没关系；
- 不要在一个文件里出现两个相同的规则；
- 用 border: 0; 代替 border: none;；
- 选择器不要超过4层（在scss中如果超过4层应该考虑用嵌套的方式来写）；
- 发布的代码中不要有 @import；
- 尽量少用 ' * ' 选择器。

```
/* not good */
.element {
}

/* not good */
LI {
    ...
}

/* good */
li {
    ...
}

/* not good */
.element {
    color: rgba(0, 0, 0, 0.5);
}

/* good */
.element {
    color: rgba(0, 0, 0, .5);
}

/* not good */
.element {
    width: 50.0px;
}

/* good */
.element {
    width: 50px;
}

/* not good */
.element {
    width: 0px;
}

/* good */
.element {
    width: 0;
}

/* not good */
.element {
    border-radius: 3px;
    -webkit-border-radius: 3px;
    -moz-border-radius: 3px;

    background: linear-gradient(to bottom, #fff 0, #eee 100%);
    background: -webkit-linear-gradient(top, #fff 0, #eee 100%);
    background: -moz-linear-gradient(top, #fff 0, #eee 100%);
}

/* good */
.element {
    -webkit-border-radius: 3px;
       -moz-border-radius: 3px;
            border-radius: 3px;

    background: -webkit-linear-gradient(top, #fff 0, #eee 100%);
    background:    -moz-linear-gradient(top, #fff 0, #eee 100%);
    background:         linear-gradient(to bottom, #fff 0, #eee 100%);
}

/* not good */
.element {
    color: rgb(0, 0, 0);
    width: 50px;
    color: rgba(0, 0, 0, .5);
}

/* good */
.element {
    color: rgb(0, 0, 0);
    color: rgba(0, 0, 0, .5);
}
```

&nbsp;

*Posted on 2018-05-26*
