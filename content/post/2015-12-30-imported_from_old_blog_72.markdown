---
date: "2015-12-30T17:09:47Z"
title: 我的 TeX 上手环境（BasicTeX）
---

最近为了写“自然科学中的混沌与分形”课程论文，决定学习一下TeX。首先需要配置TeX环境。

我的系统是OS X 10.11.3 beta，在StackOverflow上搜到不少Mac配TeX环境的教程，知乎上也有一些，大都推荐了MacTeX。它是一个2GB+的.pkg安装包，按介绍说包括了一般用得到用不到的所有相关文件，还带了图形界面编辑器。但是我比较想要从头自定义一个环境，这样用起来比较顺手还不臃肿，所以转而选择BasicTeX。它是一个最小化的TeX Live Destribution，装好大约300MB大小。<a href="http://tex.stackexchange.com/questions/97183/what-are-the-practical-differences-between-installing-latex-from-mactex-or-macpo" target="_blank">这篇回答</a>我感觉写的不错，讲到了装好BasicTex之后需要干的事。我的配置流程大致就是按照它的思路做的。

事实上安装完TeX Live之后，只需要找个编辑器就行。其实比如vim、atom之类的就可以，不过我还是想要个IDE，省的花时间搞一堆配置。最终选定了TeXShop，开源软件，而且和BasicTeX配合，无需配置环境变量之类的。感觉非常方便。

TeXShop自带一些模版，看一会儿基本就理解TeX语言的思路了，所以入门其实很简单，难的应该是熟练各种宏。要使用中文，可以安装ctex这个包。安装新包用命令行工具tlmgr，不过变态的是，tlmgr的依赖计算估计是有点问题，或者TeX社区不怎么关心这个问题，装了个ctex之后还得手动装一堆（大概3-4个）包。应该有一些工具解决这个问题，最近找一找。

感觉这套环境安装很快（下载 + 安装不到5分钟），而且无需配置，挺适合新用户。

LaTeX的文档结构，可以参考<a href="https://en.wikibooks.org/wiki/LaTeX/Document_Structure#Preamble" target="_blank">这篇文章</a>。