---
layout: post
title: 把博客迁移到了 Jekyll
date: '2017-11-01 23:40:00'
---

之前三年这个博客一直用的是Ghost。Ghost是一个非常不错的blogging平台，用node.js写的。我在它刚开始beta的时候就试用了一下，过了段时间后也从Wordpress迁移到了Ghost。说它好主要是因为：

1. 没有杂七杂八的功能（这也是Ghost项目的初衷之一）
2. 性能比Wordpress好
3. 编辑界面和后台都挺漂亮的

一用两三年，Ghost终于在去年这个时候（秋天）开始为1.0.0做准备，我当然也是比较激动，一直持续关注开发进度。直到今年初夏，1.0.0 release candidate终于发布，于是我赶紧往服务器上装。安装过程中我再次体验了npm可怕的内存占用，最高到1.5GB，而且耗时几十分钟——实在不算愉快。事实上很简单的工作——确认dependency、获取Ghost源代码、初始化数据库，在Javascript的web-scale工具链下变得复杂无比。安装之后发现，Ghost 1.0.0的管理后台仍然很不错，但是新的Casper主题在我的电脑上卡的一愣一愣的，令人惊叹13年版本的MacBook现在连上个博客都吃力了。刚好赶上暑假忙着，就暂时没管，但决定有一点空了立马换掉它。

前段时间稍微有点空，我开始考虑新的blogging platform。入围的有：Medium、Static Site Generator、Posthaven。首先排除了Posthaven，因为感觉对我来说性价比不高。排除Medium主要是想真正控制我的内容，东西放到Medium上可能就被lock-in在他的格式、API、feature里了。于是Static Site Generator（SSG）顺理成章成为唯一选择。目前比较热门的SSG有Jekyll、Pelican、Hugo和Hexo。Hexo是JS写的，我比较慌，所以果断排除。Pelican的社区感觉小了些，而且相对于另外两个看不出啥独特的feature，所以排除。Jekyll和Hugo真是各有千秋。Hugo是Golang写的，装起来来得个方便，一个binary一拷就行，没有任何dependency。Jekyll是Ruby写的，存在时间长，插件数量和community都很不错。再加上Jekyll的默认主题蛮好看的，最终选了Jekyll。

部署过程就很容易了，有人写过Ghost转换到Jekyll的脚本，过程比较顺利。目前这个站点部署在Netlify上，Netlify专门host静态站点和SSG生成的站点，可以自动provision Let's Encrypt的证书，也可以和GitHub webhook集成实现自动build，总之用起来十分方便（而且暂时免费）。之后可能会再放回自己的服务器上。
