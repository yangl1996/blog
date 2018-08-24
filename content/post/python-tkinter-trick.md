---
date: "2015-12-25T14:06:00Z"
aliases:
    - /2015/12/25/python-tkinter-trick.html
title: Python Tkinter 编程的一个小问题
---

刚刚cfw同学给我看了一个神奇的现象，类似如下一段代码：

```
def _ms(self):
ms = tk.Tk()
ms.geometry('450x250')
ms.title('AAA')
label2 = tk.Label(ms,text = 'Test')
v_next_dest = tk.StringVar()
entry2 = tk.Entry(ms,textvariable = v_next_dest)
button1 = tk.Button(ms,text = 'Next',command = lambda:print(v_next_dest.get()))
entry2.grid(row = 2,column = 1)
button1.grid(row = 3,column = 0,columnspan = 2)
ms.mainloop()
```

运行的时候，照理说按下Next按钮之后，会在CLI中输出entry2中当前的内容。这段代码做的工作是一个GUI程序的某一个窗口。但是当运行整个程序，进入这个窗口，做上述操作时，却没有达到预期的效果，而是只输出空行（相当于v_next_dest.get()没有得到东西）。更奇怪的是，如果我建立一个这个程序的实例，然后直接强行调用self._ms()（相当于直接运行这个窗口）的话，一切又正常了。

由于直接调用entry2的get()方法也可以达到类似效果，我就顺便试了试，结果这样写居然是可以正常运行的。所以问题就显得有些奇怪。我参考了上个寒假和Jueast一起写的<a href="https://github.com/B-O-P/mini-chatter" target="_blank">mini-chatter</a>代码，才找到原因。

事实上，问题的原因是，一个Tkinter程序只能有一个Tk实例，之后如果要开多个窗口，那要建立的是Toplevel实例而不是Tk实例。这两种对象用起来基本是一样的。当把函数主体第一行Tk改成Toplevel，一切正常工作。我大致猜测之前出问题估计是和tkinter库处理Textvariable的方法有关。

总之，一个Tkinter GUI程序只应该有一个Tk实例，剩下的窗口通过创建Toplevel实例来达成。