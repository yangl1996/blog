---
layout: post
title: 解决Gravatar头像不能读取对WordPress的影响
date: '2014-11-22 09:52:32'
---

今天发现博客首页的图片加载不出来，但是退出账号之后就加载的出来了，于是怀疑Gravatar出问题。打开Safari的时间线一看，发现果然是卡在Gravatar头像那里了。之前Gravatar网站不能访问，没想到现在要调用Gravatar的头像也不能了……好在已经有了这样的插件，实现 cache Gravatar 到服务器本地的功能。果断装上之后恢复正常了。

话说是不是应该好好整理一下，WordPress究竟调用了哪些外面的服务，这样突然出什么问题的话就有Debug流程了……