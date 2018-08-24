---
date: "2016-05-19T16:46:56Z"
title: 受限环境下通过串口传送二进制文件
---

最近要在学校开放日做一个实验室项目的 Demo ，需要在嵌入式板子 Cubieboard 2 上跑一个小程序。板子上跑的 Linux Kernel 和 Root FS 都是我自己裁剪并用 Buildroot 提供的交叉编译工具链编译的。其中，无比精简的 Root FS 压缩后和 Kernel uImage 链接在一起，启动时直接作为 RAM Disk 挂载为 Root 文件系统。板子和我 Mac 的唯二连接：一根电源线和一个串口。

在这个环境下，我需要把 host machine 上交叉编译出来的二进制文件放到 Cubieboard 上跑。由于可能的 Debug 过程，预期要进行很多次这样的操作。简单一想，方便的解决方案如下三种：

1. 每次编译后，让 Buildroot 重新打包并链接 Root FS ，把新文件放到文件系统某处，最后载入到 SD 卡上
2. 通过串口拷贝文件到板子上，每次掉电后会没掉，所以最后 Demo 时要用方法 1 写到文件系统上
3. 直接在板子上编译

方法三完全不用考虑。如果我把 gcc 放上去，那这些问题早就不存在了。

方法一和方法二理论上都行得通，区别是一个比较方便，适合 Debug 阶段，另一个每次都要生成 Root FS 和写 SD 卡，流程下来至少三分钟。所以我优先考虑方法二。

Cubieboard 上的环境极其受限：为了尽可能缩小文件系统体积（因为整个文件系统被拷贝到内存里），我 Busybox 用的是默认的，什么包都不装的设置。其实 StackOverflow 上通过串口传二进制的方法一抓一大把，但是都依赖各种传输协议，也就是说需要在 target machine 上预装对应协议栈或处理程序。这样一来又增大文件系统体积了。因此我需要在极其受限的环境下完成这个工作。

其实要让 host 发送二进制流太容易了，直接 `cat` 然后重定向到 `/dev/ttyUSB0` 不就好了。但是 target 得运行个进程接收啊，不然就全到 shell 里了。我先尝试 vi 但是发现 vi 一股脑儿接收大量二进制流后，会直接爆炸，连带整个 shell 也崩了。这个方法传 ASCII 完全没问题，但传二进制就要考虑 vi 处理二进制文件的能力。板子上的 vi 是没有 binary 模式的。

之后我又尝试花式重定向，比如 target 上 `cat /dev/ttyS0 > received.bin`，然后 host 上把文件 `cat` 到串口上，可惜这样 shell 还是爆炸了。其实串口本身当然可以传二进制，但问题就在数据走的是 TTY 设备，显然 TTY 只用来处理 ASCII 字符，遇到二进制流当然就爆炸了。

__下面是我最后成功的解决办法__：要避免这个问题，解决办法就是把二进制流先编码成 ASCII 流。可以用 Busybox 自带的 uuencode 和 uudecode 来实现。它们本来是用来把各类文件编码，用于 email 传文件的。

uuencode 的任务就是把二进制文件编码成 ASCII 输出。用法是 `uuencode <output-name>` 而作为一个典型的 UNIX 程序，这里的 `<output-name>` 指的当然不是保存结果的路径，而是 uuencode 在编码时，会在开头附上一个文件名。接收方解码后就以它为文件名保存。所以，我们只需要 `uuencode received.bin < tosend.bin > /dev/ttyUSB0` 就可以了。接收方会把文件保存成 received.bin 而我们发送的是工作目录下的 tosend.bin 文件。

uudecode 的任务很简单，只要解码并保存就行。把 TTY 重定向到它 `uudecode < /dev/ttyS0` 即可，它会阻塞 TTY 以等待传输，传输结束后自动退出。

这样一来二进制文件就传输好了。为了保证文件完整性，可以考虑先 tar 一下，接收后解包，顺便就验证 integrity 了。整个过程，对 target 的依赖只有 Busybox 自带的 uudecode 而已，这应该是对 target 依赖最小的解决方案了。