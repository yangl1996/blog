---
layout: post
title: GSoC Community Bonding阶段的见闻
date: '2015-06-13 17:22:33'
---

5月到6月是GSoC的community bonding时段，参加者在这段时间熟悉开源社区的工作流程，顺便和大家讨论讨论自己的项目。

这个月发了非常多的邮件，还参加了好几次IRC Meeting，和印度同学Kunaal对整个项目的workflow达成了一致，并且做了一下前期的分工，算是完成了community bonding阶段。导师方面，他回邮件比较少，但是会参加邮件列表上的讨论，我们无法表达清楚意思的时候还会赶来“救火”。

我遇到的社区里的人都非常友好，而且一起发邮件交流效率很高。邮件列表上有不少关于项目的建议和问题，还有人指出如果要涉及到第三方的平台，如何在夏天结束后将项目顺利交接给CentOS的运维人员也是个问题。法国的Pierre-Yves Chibon是Pagure的作者，我们正用Pagure作为toolchain里的一部分。Pagure是一个非常新的项目（API版本号还是0，而且没有公开API Reference，代码仓库现在还有每天2-3次的commit）。Pierre非常有趣又很热情，我发给他的第一封邮件他过了双休日才回，回复里道歉说他每隔一段时间会afk（Away from Keyboard），所以周末没上网。之后详细回答了我的问题，还把未开放的API Reference给我（其实错误非常多没法用，这也是没有公开这份Reference的原因）。

今天写一段需要用到Pagure API的代码，调用一些需要登录才行的接口。可是查看API Reference半天居然找不到如何认证，本来想发邮件问Pierre，但是后来决定自力更生，终于在Pagure的测试脚本里找到了API的认证方式，顺便还发现Reference里列举的调用参数有很多错。原来要实现的功能终于成功实现，但是这么一折腾之后，我决定帮Pierre写一份API v0的文档。给他发邮件之后他马上回复表示非常开心，还说“The more, the merrier”。

关于邮件，这段时间学到“inline reply”的技巧，就是在写回复时，直接把回复的内容插在引文中，做到对每一段分别回复。邮件的礼貌基本就是多表达感谢和赞扬。老外写邮件经常用“thanks”、“awesome”、“cool”之类的词，对抬头、署名之类的倒不是非常讲究，写邮件时基本也是直接称呼对方的名。另外关于IRC，养成了开电脑就开上IRC的习惯，这样如果社区里有人有急事就可以马上找到我。顺便推荐目前OS X和iOS上最好用的IRC客户端：Textual和Palaver。

项目具体coding方面，现在做完了github到Pagure的同步，Kunaal部署了一个CI，用来实时build文档页面。这两步完成之后，项目的基础基本就OK了，可以说是实现了最基本的功能。下一步考虑的是如何把文档推送到相关的开源项目。