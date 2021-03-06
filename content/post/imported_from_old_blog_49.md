---
date: "2014-11-21T05:58:55Z"
aliases:
    - /2014/11/21/imported_from_old_blog_49.html
title: 计算概论A大作业：扫雷
---

本学期计算概论A的期末大作业是制作一个扫雷。上个礼拜六闲着无事，花了整个晚上做完了基础的功能，后来又添加了一些特性，现在已经基本完成了。代码总长度1000行左右。

开始做的时候大作业的要求还不明确，所以一开始只是打算试一试，也没打算一晚上做完，结果做到第二天早上五点就做完了。

作为命令行下运行的程序，UI本身已经十分简陋了，交互方式不能再很麻烦。所以考虑实时反馈用户的按键操作。Mac OS中要实现实时读取输入其实是很麻烦的。本来一直想用getch()这个函数，无奈相关的库调用之后总是报错，最终在网上找到了另外的实现，用的都是标准库函数。另外关于清屏。每次读取输入之后都需要清屏然后重绘整个界面。清屏用了system函数，牺牲了一些可移植性。不过如果代码要在Windows下编译运行，要修改的内容也不多。还有读取输入之后的判断之类琐碎的事情，比如对arrow key的读取和判断之类，稍稍查找资料也很容易解决。

![](/content/images/2016/05/codesplit1.jpg)
Mac OS下用于不回车读取输入的代码

这次的失误之处在于没有精心设计数据结构。因为一开始只是打算试一试，所以各种变量，包括变量名，都没有预先设计好，而是要用的时候临时添加。完全写完之后再读代码，就发现有相当多冗余成分，有的变量命名也晕头转向。不过这个时候要修改的话成本实在太大，于是只能放着。具体体现在，描述棋盘状态居然用了6个数组，现在看看其实用3个也够了。在添加存盘读盘功能的时候，因为之前乱开变量，极其头痛。传入的参数居然有14个。为了安全，这么多的变量不敢全部设成global，所以读盘的时候得吧代码全部写在主函数里面，结果就是主函数非常长，大概有700行吧。现在想想要是一开始先设计好数据结构，甚至直接用面向对象的方法来写，那后期应该会舒服多啦。

写完扫雷的主体部分之后，开始添加各种杂七杂八的功能，包括彩色输出、存盘读盘、计时器、主菜单等等。彩色输出用到了ANSI控制码，存盘读盘用到fstream对象，基本都是现学现卖。彩色输出很重要，要不然全是黑的，根本看不清楚，光标和雷都分不出来。测试合理的配色搞了好一会儿。计时器现在做到在赢的时候输出一个耗时，想想如果不能记下来的话也没什么意义，所以打算再做一个排行榜。主菜单几经润色，现在也不错了。

至于扫雷主程序的算法。输出棋盘的部分写了几个函数，主要就是鼓捣制表符。翻开棋盘的部分是一个递归，这个递归改了好长时间都觉得不对，结果发现是因为自己根本就不会玩扫雷……拼小命玩出一局扫雷之后，递归函数基本改对了。这个递归难度不大，基本上和“孙悟空找师傅”差不多难度。不过写的时候发现缺了几个变量，于是临时补了好多，代码违和感有点强……（以后一定要先设计数据结构，不能总是面向过程）布雷的部分，一开始用了极为愚蠢的算法，后来也懒得改了。难度设定部分，深深体会到程序的容错性有多重要，一开始随便按错一个键就直接死循环，现在好点了。

整个编写过程用Git做版本控制，高枕无忧。不过有一次增加新Feature的时候开一个branch，然后分别在master和feature分支里面commit了，想merge的时候发现conflict。还有一次改动了feature分支，但是忘记了commit就直接merge了，做的改动全部丢掉……GitHub一如既往地好用，真是越来越好用了。这次玩了一下他的Release功能，到现在为止一共发布了8个Release。

这个作业做完还是有不少收获的，最大一点的就是，对于稍微长一些的程序，都应该先设计数据结构，而不能直接开始写。也算是实践了一下稍稍复杂的shell控制和文件操作。

---

写好了排行榜功能，并且把上文提到的Bug改掉了。现在代码长度是1500行。应该还是有很多冗余。不过现在要改动的话投入就很大了，划不来。

代码和二进制文件在GitHub上，[yangl1996/mine-sweeper](https://github.com/yangl1996/mine-sweeper)