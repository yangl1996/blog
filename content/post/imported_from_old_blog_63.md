---
date: "2015-05-07T16:21:13Z"
aliases:
    - /2015/05/07/imported_from_old_blog_63.html
title: 如何回复邮件列表中已存档的邮件
---

之前回邮件列表的时候总是手动复制主题，前面加一个"Re: "然后直接发出去，但是总被Mailman认为是一个独立的主题。今天稍微找了下解决办法，其实极其简单：

![](/content/images/2016/05/figure.jpg)

点一下归档的邮件标题下面的anti-spam邮件地址，事实上是一个mailto链接，链到邮件列表的地址（而不是显示的发件人地址）。然后在邮件客户端里面直接回复就ok。

当然啦，最好GNU Mailman也做一个类似Google Groups那样的Web界面，毕竟现在不少用户从未接触邮件列表。