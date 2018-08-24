---
date: "2015-02-10T14:23:59Z"
aliases:
    - /2015/02/10/imported_from_old_blog_60.html
title: OS X下Python程序的打包发行
---

和[Jueast](http://jueast.com)一起做的[FrogTalk](https://github.com/b-o-p/mini-chatter)肝了一晚上愉快地完成了初始版本，需要打包便于分发！大部分打包软件并不支持Python 3.x，只有cx\_Freeze和[py2app](http://pythonhosted.org/py2app/)提供了支持。cx\_Freeze可以将程序打包成一个文件夹，内含dependencies和可执行文件，但是这样一堆零散的文件看起来并不好。py2app则很不错，直接打包成.app文件，双击即可执行。

首先通过pip安装py2app：

```
python3 -m pip install py2app
```

然后先生成打包配置文件：

```
py2applet --make-setup MyApplication.py
```

接下来打包即可：

```
python3 setup.py py2app
```

这样就生成了直接可执行的.app文件！

下面是几种自定义的内容：

#### 图标
用icns格式的图标。icns格式可以在[easyicon](http://www.easyicon.net/covert/)在线转换。

接下来在生成的setup.py配置文件中，
`OPTIONS = {'argv_emulation': True}`
换成
`OPTIONS = {'argv_emulation': True, 'iconfile':'icon.icns'}`

#### Info.plist
主要用于描述程序的一些信息。

* `CFBundleGetInfoString`是显示在Get Info窗口内的字串
* `LSBackgroundOnly`表明这是否是一个后台程序
* `LSUIElement`的程序不会在Dock中驻留，但仍然能到前台来显示UI

像这样作为字典的另一个键值对，加在icon的后面就行：

```
'plist': {'CFBundleShortVersionString': '0.1', 'CFBundleGetInfoString': 'Frog Talk for Mac'}
```

如果遇到缺少tcl frameworks的话，需要去ActiveState下一个tcl 8.5的框架。