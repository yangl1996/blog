---
layout: post
title: 解决https时对useso公共库的引用问题
date: '2014-09-02 17:45:05'
---

我在WordPress的登录页面和后台使用https。但是由于useso的公共库不支持https访问，所以每次打开的时候很困扰，也就是说字体和ajax库加载不出来。

今天很幸运发现了中科大提供了https可用的公共库镜像。所以对插件略作修改：

```
$str = str_ireplace('//fonts.googleapis.com/', '//fonts.useso.com/', $str);
$str = str_ireplace('//ajax.googleapis.com/', '//ajax.useso.com/', $str);
```

改为

```
$str = str_ireplace('//fonts.googleapis.com/', '//fonts.lug.ustc.edu.cn/', $str);
$str = str_ireplace('//ajax.googleapis.com/', '//ajax.lug.ustc.edu.cn/', $str);
```

一切ok了！访问速度也不错的。