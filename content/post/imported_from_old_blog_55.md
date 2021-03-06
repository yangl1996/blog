---
date: "2014-12-07T17:05:36Z"
aliases:
    - /2014/12/07/imported_from_old_blog_55.html
title: 前缀表达式
---

前缀表达式又称为波兰表达式，将运算符提前，就成了前缀表达式。前缀表达式是不需要括号的。

一个例子：(2 + 3) * 4 就是 * + 2 3 4

程序的任务就是，写一个前缀表达式的解析器。

看到题目我想到的先是设置数值栈和运算符栈，然后依次弹栈计算，结果压回数值栈。后来又想到一个更加简洁的方法：不要一口气读入整个式子，而是在递归函数中一次次读。递归函数 calc() 先读入一节表达式，如果是加号运算符，就返回 calc() + calc()，其他运算符类推。如果是数值，那直接返回就行。这样一来就避免了复杂的数据结构。

由此联想到前段时间看的《布尔表达式》问题。题目就是要求写一个布尔表达式的解析器。由于这道题目帮老师录制了讲解视频，之前看了三种解法，有一种就是设置数值栈和运算符栈，然后逐个弹栈计算。

需要考虑的问题是：为什么在波兰表达式中，可以轻松避免堆栈的运用呢？我认为，这是由于波兰表达式实在是很简洁，主要是运算顺序就是运算符出现的顺序，可以直接开始递归，而且不存在括号之类，所以计算它的步骤相当简单。相对的，布尔表达式有括号，而且存在运算优先级问题（先出现的运算符不一定能够先去计算），所以用堆栈进行存放是很好的方法。