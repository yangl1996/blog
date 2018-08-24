---
date: "2012-05-25T10:22:33Z"
aliases:
    - /2012/05/25/imported_from_old_blog_15.html
title: 关于Arduino各个库中write与print的区别
---

<blockquote>这个问题困惑了我一段时间……</blockquote>
Arduino有许多库（特别是原生的库）中有write和print函数，还有println。比如Ethernet库的client类、LiquidCrystal库，等等。

有一段时间我一直弄不零清。现在终于理解了。总结如下

write——给Arduino或Shield发送命令，相当于控制那些硬件
print——让硬件做出发送的动作，比如
<code>client.print("hello, world!"); </code>
就是让它做“发送”那件事（怎么表达好呢……）
println——在print以后在换行，我自己主要用在需要输出变量时（通过这个把一句话分成几段发）和一段要发送的东西结束时，看看这个
<code>client.print("Temperature is");
client.print(t);
<strong>client.println</strong>("over");</code>