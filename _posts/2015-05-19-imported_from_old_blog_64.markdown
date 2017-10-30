---
layout: post
title: 超长输入输出程序的Debug
date: '2015-05-19 13:52:23'
---

刚刚做到一题输入输出都很长的题目。OS X的tty会自动在1024字符处切断，所以用Xcode或Terminal都没法正常debug，又不想切Linux虚拟机，所以非常讨厌。好在StackOverflow上找到了如下的解决办法：把输入输出流重定向到文件。

```
if (!freopen("/Users/yangl1996/Desktop/string.txt","r",stdin))
{
cout &lt;&lt; "error opening" &lt;&lt; endl; // handle exception
}
```

就可以实现从文件读入，用到的是stdio.h而不是fstream。不需要更改已经写好的程序的流对象，相当于直接重定向了cin、cout。目测速度也是极快的。

同样地，可以把输出也重定向到文件，只要用w方式打开文件，并把stdin改成stdout。这样的好处是方便比较你的输出和标准输出。输入输出内容都已经存在文件里，一个diff命令就ok。

顺便记下一招让iostream提速十倍的方法：

```
ios::sync_with_stdio(false);
```

这样一来关闭了iostream与标准C输入输出的同步，可以加快速度。不过用了这招之后就不能同时使用iostream和stdio了。