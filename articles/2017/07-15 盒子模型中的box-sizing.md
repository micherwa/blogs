本文将介绍 CSS 概念中盒子模型的 box-sizing 属性的用法，它已被广泛应用于移动端布局。

在写一个块级元素的时候，经常会碰到一个问题：盒子模型。

当你设置了元素的宽度，实际展现的元素却超出你的设置：这是因为元素的边框和内边距会撑开元素。

例如：
```
.simple {
    width: 500px;
    margin: 20px auto;
}
.fancy {
    width: 500px;
    margin: 20px auto;
    padding: 20px;
    border-width: 10px;
}
```

结果如下：

![bg-20170715-01.png](https://github.com/micherwa/blogs/blob/master/images/2017/bg-20170715-01.png)

如果想让这两个 div 看上去一样宽，按传统的办法，需要减去内边距和边框，重新定义 width。值得庆幸的是，现在不需要再这样做了...

传统的盒子模型并不直观，所以css3后来新增了 `box-sizing` 属性。

当你设置一个元素为 `box-sizing: border-box; `时，此元素的内边距和边框不再会增加它的宽度。这里有一个与前一页相同的例子，唯一的区别是两个元素都设置了 box-sizing: border-box;

改一下上面的代码：

```
.simple {
    width: 500px;
    margin: 20px auto;
    box-sizing: border-box;
}

.fancy {
    width: 500px;
    margin: 20px auto;
    padding: 20px;
    border-width: 10px;
    box-sizing: border-box;
}
```

效果如下：

![bg-20170715-02.png](https://github.com/micherwa/blogs/blob/master/images/2017/bg-20170715-02.png)

如果想让页面上所有的元素都按这种更直观的方式进行排版，可以这样：

* {
    box-sizing: border-box;
}

兼容性上，box-sizing 支持 ie8+，最好再加上-webkit-和-moz-的前缀。

下面这张图更形象地对 box-sizing 的 `content-box` 和我们喜爱的 `border-box`，做了对比的效果：

![bg-20170715-03.png](https://github.com/micherwa/blogs/blob/master/images/2017/bg-20170715-03.png)

但需要注意的是，并非任何时候都需要 border-box，还是要从实际需求出发。

&nbsp;

*Posted on 2017-07-15*
