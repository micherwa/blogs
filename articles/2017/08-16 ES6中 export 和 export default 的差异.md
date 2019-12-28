**共同点：**
- export与export default均可用于导出常量、函数、文件、模块等。

**区别：**
- 在一个文件或模块中，export、import可以有多个，export default仅有一个
- 通过export方式导出，在导入时要加{}，export default则不能加

直接看下面的例子：

`export`

```
//demo1.js
export const str = 'hello world’;
export function f(a) { return a+1; };
```

对应的导入方式：

```
//demo2.js
import { str, f } from 'demo1'; //也可以分开写两次，导入的时候带花括号
```

`export default`

```
//demo1.js
export default const str = 'hello world';
```

对应的导入方式：

```
//demo2.js
import str from 'demo1'; //导入的时候没有花括号
```

#### 总结

**使用export default命令，为模块指定默认输出，这样在import的时候，可以自定义该模块的名字，不需要知道模块本身的变量名。**


&nbsp;

*Posted on 2017-08-16*
