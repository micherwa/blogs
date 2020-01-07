## 前言

今天为大家分享一款我用了很久的编辑器：sublime text 3。它的轻便和高度可定制化，是我觉得最棒的地方。本文会涉及到 sublime text 3 的方方面面，方便自己的同时，也希望能帮到你。

&nbsp;

## 安装与初始化

首先，可以在 [官网](http://www.sublimetext.com/3) 下载最新版的安装文件。选择对应的平台，我用的是 mac 版，最新的安装包也才 15.2 M，真的很轻量了。

安装完成后的第一件事，就是按照自己的习惯进行定制啦~

打开 Preferences->Setting，会分屏显示目前的配置。左侧为 Setting-Default 的系统默认配置，不需要动它。一般我们会在右侧的 Setting-User 中添加定制化的配置，我的配置表如下：
```
{
    // 默认使用Unix换行符，统一风格
    "default_line_ending": "unix",
    // 编写代码时，右侧的代码地图的可视区域部分是否加上边框
    "draw_minimap_border": true,
    "font_face": "monaco",
    // 显示的字体大小
    "font_size": 14,
    // 突出显示当前光标所在的行
    "highlight_line": true,
    "line_padding_bottom": 1,
    // 设置每一行到顶部，以像素为单位的间距，效果相当于行距
    "line_padding_top": 1,
    "tab_size": 4,
    // 使用空格填充 tab 键
    "translate_tabs_to_spaces": true,
    // 保存文件时会删除每行结束后多余的空格
    "trim_trailing_white_space_on_save": true,
    // 关闭自动更新检查
    "update_check": false,
    // 自动换行
    "word_wrap": "true"
}
```

&nbsp;

## 快捷键

快捷键的熟练使用，可以提高你的编码效率。sublime 的快捷键有很多，下面列举的这些常用的快捷键，希望能给还不太熟悉的童鞋一些帮助。

![bg-20190628-01.png](https://github.com/micherwa/blogs/blob/master/images/2019/bg-20190628-01.png)

&nbsp;

## 插件

想要优雅地使用 sublime，插件则是不可缺少的存在。强大的插件功能，造就了它无与伦比的扩展性。

在安装插件之前，首先需要安装 Package Control 组件（它本身也是个插件哦~）。

点击 sublime 的菜单栏 view->show console，打开控制台，然后运行以下代码：

```
import urllib.request,os; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); urllib.request.install_opener( urllib.request.build_opener( urllib.request.ProxyHandler()) ); open(os.path.join(ipp, pf), 'wb').write(urllib.request.urlopen( 'http://sublime.wbond.net/' + pf.replace(' ','%20')).read())
```

等待几秒钟后，如果能在下图中看到 Package Control，就表示安装成功了。


![bg-20190628-02.png](https://github.com/micherwa/blogs/blob/master/images/2019/bg-20190628-02.png)

安装插件也十分简单，点开上图的 Package Control，直接搜索你想安装的包即可。


![bg-20190628-03.png](https://github.com/micherwa/blogs/blob/master/images/2019/bg-20190628-03.png)


接下来，会列举一些非常实用的插件，并简单介绍其各自的使用场景。

#### Material-Theme

个人比较喜欢的主题，本文的截图用的正是这个主题。

用法：下载成功后，只需要修改 Preferences->Setting 的 Setting-User，引入主题。具体配置如下：

```
{
    // 引入下载的主题包
    "color_scheme": "Packages/Material Theme/schemes/Material-Theme.tmTheme",
    // 设置主题
    "theme": "Material-Theme.sublime-theme",
    // 用来将已修改的代码的标签高亮
    "highlight_modified_tabs": true,
    "ignored_packages":
    [
        "Vintage"
    ],
    ...
}
```

#### babel

用于 react 的语法高亮。

安装后，打开 .js, .jsx 后缀的文件，打开菜单 view， Syntax -> Open all with current extension as... -> Babel -> JavaScript (Babel)，选择 babel 为默认 javascript 打开 syntax。

#### SCSS

支持 sass 语法提示。

相比于 SASS，SCSS 更适合 .scss 的语法支持。因为 SASS 在某些情况下会被其他的插件影响而导致语法提示失效，这时可以引入另一个插件 Syntax Highlighting for Sass，来解决这个问题。

但如果直接用 SCSS，则不会出现这个问题。

#### html5

支持hmtl5规范的插件包。

使用方法：新建 html 文档 > 输入 html5 > 敲击 tab 键 > 自动补全 html5 规范文档。

#### git 和 git blame

安装 git 可以在底部状态栏显示当前文件的 git 状态。

而 `git blame` 则用于查看 git 记录。**很赞的一点**是，它可以查看每一行代码在 git 上的记录，包括 提交人、提交时间以及标识。

#### docblockr

快速帮你为代码建立文档。它会解析函数，变量，和参数，根据它们自动生成文档，你只需要去填充对应的说明即可。

用法：'/**' 快捷键能快速生成函数注释。

#### EditorConfig

控制代码的缩进等的代码规范插件。

安装好之后，在工程中创建文件.editorConfig，例如：

```
# editorconfig.org
root = true

[*]
indent_style = space
indent_size = 4
end_of_line = lf
charset = utf-8
trim_trailing_whitespace = true
insert_final_newline = true
[*.md]
trim_trailing_whitespace = false
```

#### Bracket Highlighter

可匹配 []、()、{}、""、''、<tag></tag>，高亮标记，便于查看起始和结束标记。

#### SideBarEnhancements

增强 sublime 的右键功能，包括拷贝、粘贴、删除、重命名等。

#### AutoFileName

可以自动补全文件名，如图片选取。比如：输入 "/" 即可看到相对于本项目文件夹的其他文件。

#### ConvertToUTF8

将文件转码成 UTF-8。用于编辑并保存目前编码不被 Sublime Text 支持的文件，特别是中日韩用户使用的 GB2312，GBK，BIG5，EUC-KR，EUC-JP ，ANSI等。安装插件后自动转换为 UTF-8 格式。

#### GBK Support

可识别 UTF-8 格式的中文，不识别 GBK 和 ANSI，因此打开很多含中文的文档都会出现乱码。可以通过安装插件 GBK Support，来识别 GBK 和 ANSI。

用法：打开一个 GBK 的文件，直接用 GBK encoding 模式另存为即可。

#### FileDiffs

强大的比较代码不同工具。用于比较当前文件与选中的代码、剪切板中代码、另一文件、未保存文件之间的差别。可配置为显示差别在外部比较工具，精确到行。

用法：右键标签页，出现 FileDiffs Menu 或者 Diff with Tab… 选择对应文件比较即可。

&nbsp;

## 下载插件被“墙”？

sublime text 3 很好用，但也会有些小问题。比如，国内用户在使用 sublime 下载插件时，可能会出现 `There are no packages available for installation` 的问题。

这个问题出现的原因很简单，就是获取 Sublime Text 3 的网络地址被“墙”了。

对于有梯子的童鞋，很容易解决。但对于更多的翻不了墙的童鞋，为了给予方便，我找了一种解决方案。说白了也简单，就是使用别人能够访问的地址就行了呗，比如 github。
```
https://raw.githubusercontent.com/SuCicada/channel_v3.json/master/channel_v3.json
```

该如何使用呢？

在 "Preferences -> Package Settings -> Package Control -> Settings User" 中新添加一个项 "channels"。

```
"channels":[
    "https://raw.githubusercontent.com/SuCicada/channel_v3.json/master/channel_v3.json"
    ],
```

然后，快去下载插件试试吧~

&nbsp;

## 总结

sublime 用了挺久了，因为感觉很棒，正是我喜欢的风格。分享给有需要的朋友~

&nbsp;

**PS：欢迎关注我的公众号 “超哥前端小栈”，交流更多的想法与技术。**

![wechat qrCode](https://github.com/micherwa/blogs/blob/master/images/wechat_qrCode.jpg)

&nbsp;

*Posted on 2019-06-28*