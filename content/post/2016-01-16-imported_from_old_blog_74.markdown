---
date: "2016-01-16T23:07:48Z"
title: 在Linux上从源码安装和使用C库
---

为了准备超算竞赛，我被分配在实验室的cluster上跑通RNA序列构建工具Trinity（SC15 SCC题目）。虽然cluster跑的是CentOS，但是可惜我没有root权限，所以在解决各种依赖时稍稍花了点功夫，所有依赖都必须从源码编译，而且要自行配置。

由于无法修改/usr/lib，所以我只好把要用到的库全部装在自己的用户目录下。在make install或configure时，加上--prefix=$HOME就可以做到。如此一来，我的用户目录下有了lib/，include/，bin/，man/，share/。第一个存放库的ELF文件，第二个存放头文件，后面三个和Linux根目录下这些目录的作用一样。

由于库没有安装在“标准”位置，所以gcc和ld一般来说会报错表示找不到文件。设置这些环境变量可以解决：$CPATH=$HOME/include，$LIBRARY_PATH=$HOME/lib，第一个相当于gcc -I选项，定义include时的搜索路径，第二个相当于ld -L，定义linking时的搜索路径。为了避免每次启动bash都要设定一次，可以把环境变量添加到.bashrc里，如果一个环境变量有多个值，用冒号分隔。如果乐意，当然也可以改Makefile，或者在用gcc时自己用-I，-L指明搜索路径，不过都不方便。对于有的库，可能还要多添加几个路径。

顺便提到ncurses这个库。它是UNIX curses库的开源版本。有的Makefile会写-lcurses其实是不对的，因为Linux里用的是ncurses，只是有些发行版为了兼容legacy code，所以创建了叫做libcurses的软链接，链接到libncurses。而另外发行版，或者像我从源码编译安装，就不会有这样的链接。此时，需要把Makefile里所有-lcurses改成-lncurses。另外，早期libncurses和libtinfo是分开来两个库，需要分别链接，现在libtinfo已经包含在libncurses里了，但是有些Makefile依然会-ltinfo，这时，如果已经-lncurses，那直接把前者删掉就行，不然就改成-lncurses。

和pip install、gem install、yum install之类相比，从源码编译安装C库就像黑魔法。前者无比方便，也是我实际使用中的首选，但是后者才是开源的本质。即使只是自己改个小bug，这种自由也是其他任何方式无法提供的。改好Makefile，敲make回车的时候，多少还是能感受到现在仅存在于各大mailing list的hack精神。