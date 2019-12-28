## 前言

当浏览器检测到非用户操作产生的新弹出窗口，则会对其进行阻止。

因为浏览器认为这不是一个用户希望看到的页面。另外可以发现，当 window.open 为用户触发事件内部或者加载时，不会被拦截。而一旦将弹出代码移动到 ajax 或者一段异步代码内部，马上就出现被拦截的表现。

但是，我们仍然会遇到某些需求，当满足某些条件之后，自动触发打开新页面的操作。那么，该怎么办呢？

&nbsp;

## 解决方案

#### 1.使用a标签替代

```
function newWin(url, id) {
    var a = document.createElement('a');
    a.setAttribute('href', url);
    a.setAttribute('target', '_blank');
    a.setAttribute('id', id);
    // 防止反复添加
    if(!document.getElementById(id)) document.body.appendChild(a);
    a.click();
}
```

将此函数绑定到 click 的事件回调中，就可以避免大部分浏览器对窗口弹出的拦截了。

#### 2.先弹出窗口，然后重定向

这其实是一种变通方案，核心思想是：先通过用户点击打开页面，然后再对页面进行重定向。

```
xx.addEventListener('click', function () {
    // 打开页面，此处最好使用提示页面
    var newWin = window.open('loading page');

    ajax().done(function() {
        // 重定向到目标页面
        newWin.location.href = 'target url';
    });
});
```

以上方法其实是打开了两个地址，为了避免打开两次真正的目标页面，让用户察觉到页面的重定向，可以在打开第一个地址的时候给出一个类似‘当前页面正在加载中，请稍后。。’的简单提示页。