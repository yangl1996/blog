---
date: "2015-11-28T19:02:22Z"
aliases:
    - /2015/11/28/imported_from_old_blog_70.html
title: libpagure 进入 Fedora Package Repository 了
---

<a href="http://www.yangl1996.com/libpagure/" target="_blank">libpagure</a>是我暑假参加GSoC时新建的项目。起因非常简单，我写的GSoC项目需要大量调用<a href="https://pagure.io" target="_blank">Pagure</a>的API，所以就跟创建者Pierre联系，开了libpagure这个坑。Pagure的API不多，所以我花了一晚上用简单的OOP封装了一下就完成了。完成之后基本就没再动过，后来和Pierre邮件里说起要写一组单元测试，因为开学马上就很忙所以也一直没填这坑。

月初忽然收到Chowdhury发来的邮件，说给libpagure提交了一个Pull Request但我一直没回复，才发现确实有个PR放着一周没处理。邮件里说他做一个项目用到Pagure，也要用到libpagure，顿时又开心又害怕，开心觉得自己的项目被人用上了，害怕觉得自己花一晚上写的代码肯定有不少问题。处理完之后Pierre回邮件问我是不是很忙，有没有空管理这个项目，如果没空的话愿意帮我一起管。我马上把他加进Repository Admin，回邮件时候顺便赞扬了下这学期占用我不少时间的ICS课。

这两天忽然想起这件事，google了一下，看到Redhat的bugzilla里有人讨论libpagure的bug，才发现Pierre他们已经打包好libpagure发布到Fedora Package Repository。看来寒假还是要把这个坑好好地填好，不然Fedora用户装上之后出一堆bug就太不好了 0.0