---
title: "迁移到Hugo"
date: 2018-08-24T15:55:00-04:00
---

在把这个博客[从Ghost迁移到Jekyll](http://blog.yangl1996.com/post/move-to-jekyll/)后不到一年，我又把它迁移到了Hugo。其间由于各种忙，虽然记下了很多想写的内容，但是实际上只写了一篇post。

迁移的动机如下：

- Hugo是用Go写的，装起来比较方便
- 我会写Go，但没写过Ruby（Jekyll是用Ruby写的）
- 我的[Homepage](http://leiy.me)在几个月前改用了Hugo，希望两个网站可以用同一套工具
- Hugo的build速度快
- Hugo支持relative URL，Jekyll不支持，而且[项目leader不想支持](https://github.com/jekyll/jekyll/issues/6360)

迁移过程比较简单。首先`hugo import jekyll`可以自动把frontmatter里的时间戳等信息转成Hugo使用的格式，并把一个Hugo站点最基本的结构放好。然后，写了一个脚本给所有导入的post添加alias，保证原来的链接不会坏，并把post的文件名里的时间删掉。最后，修改Netlify的build设置，让它改用Hugo build这个站点。

希望这次搬完以后可以不用再搬来搬去了，专心写两篇post……
