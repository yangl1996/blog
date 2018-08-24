---
date: "2015-07-16T17:15:54Z"
title: 在 PyPI 上发布 Python Package
---

GSoC 的项目中用到很多 Pagure API 的调用。我把写一个 API Library 的想法告诉 Pierre 后，他马上鼓励我动手写，并提醒我实践 "publish early, publish often" 的开源软件开发之道，"get people excited and let them work for you"。

实际的编写过程非常快，因为 Pagure 现在的 API 也就 10 几个左右。关于发布，我本来打算就在 GitHub 上搞一个 Release 了事，但后来想想自己做 GSoC 项目也要用到这个库，所以决定发布在 PyPI 上以便于我和印度同学在自己的服务器上安装这个库。

在发布过程中遇到一些奇怪的小问题，连同发布的全过程一并记在这里。

### Assumption

本文假设参考者满足以下条件：

* 有一个准备发布的 Python package，名为 poipoipoi
* 使用 GitHub 托管代码
* 运行 Linux 或 OS X

### 注册 PyPI 账户
在 [PyPI](https://pypi.python.org/pypi?%3Aaction=register_form) 和 [TestPyPI](https://testpypi.python.org/pypi?%3Aaction=register_form) **分别**注册一个账户。这两个看上去是一个网站，其实账号是分开的。我就因为这个被坑了相当长时间……

### 在本地配置你的 PyPI 账户

在你的用户目录创建`.pypirc`文件，也就是在`~/.pypirc`。

```[distutils] # this tells distutils what package indexes you can push to
index-servers =
pypi
pypitest
[pypi]
repository: https://pypi.python.org/pypi
username: {{your_username}}
password: {{your_password}}
[pypitest]
repository: https://testpypi.python.org/pypi
username: {{your_username}}
password: {{your_password}}
```

`.pypirc`文件的格式和以前版本的不同，所以一些网上的文档讲的可能是有误的。

密码用明文存储，所以需要做好对应的防备措施（非常建议使用独立的密码）。

### 检查 Package 的目录结构

```
poipoipoi/ # working directory
----setup.py
----setup.cfg
----LICENSE.txt
----README.md
----poipoipoi/
--------__init__.py
--------poi.py
```

除了`poi.py`，每个文件都必不可少。`poi.py`代表这个 Package 中除了`__init__.py`之外其他所有文件。

### 准备`__init__.py`

这是 import 这个 module 之后，Python 自动执行的文件。因此，如果你写的是一个库的话，就需要在这个文件中把所有对象都 import 一遍，这样 Python 解释器就可以用了（不然 Python 解释器是看不到这些对象的）。一个最简单的写法：

```
from .poi import *
```

这样就把同一个目录下`poi.py`文件中所有内容 import 了一遍，之后就可以在 Python 解释器中被调用了。

`__init__.py`的变化相当多，可以看看类似的 Package 的 `__init__.py`是怎么写的，然后照样子写一个。

### 准备`setup.cfg`

```
[metadata]
description-file = README.md
```

用来让 Python 知道 readme 文件的位置。默认的是 `README.rst`，但是 GitHub 默认的格式是 MarkDown，所以这里得注明。

### 准备 setup.py

**缩进被 WordPress 吃掉了，请不要直接复制！**

```
from distutils.core import setup
setup(
name = 'poipoipoi',
packages = ['poipoipoi'], # the same as 'name'
version = '0.1',
description = 'Poi?',
author = 'Lei Yang',
author_email = 'i@yangl1996.com',
url = 'https://github.com/yangl1996/poi', # link to your github repo
download_url = 'https://github.com/yangl1996/poi/tarball/0.1', # the last component is the same as 'version'
keywords = ['testing', 'logging', 'example'], # keywords of the project
classifiers = ['License :: GPL'], # used for classification
)
```

这里有一项`download_url`，指向的是 GitHub 上对一个 tag 版本的链接。所以，你需要手动创建这个 tag。比较方便的方法是在 GitHub 上创建一个相同版本号的 Release。

### 准备`LICENSE.txt`
把 LICENSE 加个扩展名就好。

### 发布
#### 测试用

```
python setup.py register -r pypitest
python setup.py sdist upload -r pypitest
```

#### 生产用

```
python setup.py register -r pypi
python setup.py sdist upload -r pypi
```

大功告成！