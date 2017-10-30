---
layout: post
title: C++：读取用符号分割的一串数
date: '2014-09-28 07:04:59'
---

####UPDATE

之前的方法实在太繁琐，要解决这个问题并不需要那样大动干戈。

cin可以自动地在不匹配的类型处停下。就是说比如，要通过cin将键盘输入读入int型变量，而缓冲区的内容是`100,200,300,400,`则`cin`会在第一个逗号处自动停下，然后把`100`赋给变量。这时用`cin.get()`吃掉逗号，就可以实现需要的需求。

如果要读取的是字符串，那可以使用`getline`函数。`getline`函数的第三个参数就是用于判断的分隔符。具体调用是：

```
cin.getline(char_type *__s, streamsize __n, char_type __dlm)
```

第三个参数填入需要的分隔符，比如`cin.getline(str, 30, ',')`，就可以实现自动分割。

---

在做PG上面一道练习题的时候，需要用到这个技巧。

输入的内容是一串整数，两个数之间用逗号隔开。例如：1,2,3,4,5,6

之前做到的都是用空格或者换行符分开的数据，所以可以直接`cin`的时候自动分开。另外，也会给出数据的数量，便于设计循环。这道题目则不同，首先需要设计方法来用逗号分割每个数据，然后还要判断输入的数据何时结束，以便设计循环进行存储。

首先解决第一个问题。输入的内容考虑用`string`进行存储，因为`string`有很多方便的方法。比如`replace`，可以替换其中的字符。将输入的内容存入一个`string`，然后用`replace`将其中的逗号替换为空格，这样就符合了可以用cin读取的格式。

```
string input;
cin &gt;&gt; input;
replace(input.begin(), input.end(), ',', ' ');
```

接下来，需要将这个字符串通过`cin`读取。由于`cin`只能读取流，所以这里先将这个字符串转换为一个`stringstream`。

```
stringstream ss;
ss.str(input);
```

这就生成了一个流，名字是`ss`。这样，第一个问题基本就解决了。

下面判断数据何时输入完毕。可以用while。在while的条件中用`cin`读取，如果没东西读了就会结束循环。

```
int data[1000]; // init一个数组用来存储读入的数据
int count = 0; // 一会儿用作数组的index
while (ss &gt;&gt; data[count]) //在条件中读取数据
{
    count++; // 递增index
}
```

这样，结束时，`count`表示读入的数据的数量，所有数据存在`data`数组中，可以比较方便地调用。

实现上面的操作依赖`sstream`和`string`库。

我把完整的片段放在了GitHub Gist上。[https://gist.github.com/yangl1996/1f85361ec17002cd9f4f](https://gist.github.com/yangl1996/1f85361ec17002cd9f4f)