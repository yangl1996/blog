---
layout: post
title: Mac 上 dd 的性能
image: "/content/images/2016/06/IMG_0790.jpg"
date: '2016-06-29 15:15:00'
---

UNIX 和 Linux 的 `dd` 在无数地方都要用到，比如给一个磁盘创建映像、装系统的时候写一个 Bootable USB Device 之类的。

在 Mac 上 `dd` 到外置设备的标准方法是，先 `diskutil list` 出目标的设备文件（比如是 `/dev/disk2`），然后 `dd of=/dev/disk2` （务必不要弄错设备文件！！）就可以了。但这样速度非常之慢。我前两天要给实验室一个机器装系统，写一个 4GB 的镜像要将近一小时，很痛苦。幸运的是，我在 StackOverflow 的一个答案的一个评论里遇到了解决办法。

这个方法极其简单，就是把原来的设备文件 `/dev/disk2` 换成 `/dev/rdisk2` ，再制定一个合理的 block size 。比如，用 `dd of=/dev/rdisk2 bs=1m` 可以达到原来 20 到 40 倍的速度。

为什么会这样？原因已经在 `hdiutil` 的 manpage 里解释得相当清楚了，在 Device Special Files 一节里。

> Since any `/dev` entry can be treated as a raw disk image, it is worth noting which devices can be accessed when and how. `/dev/rdisk` nodes are character-special devices, but are "raw" in the BSD sense and force block-aligned I/O. They are closer to the physical disk than the buffer cache.  `/dev/disk` nodes, on the other hand, are buffered block-special devices and are used primarily by the kernel's filesystem code.

> It is not possible to read from a `/dev/disk` node while a filesystem is mounted from it, but anyone with read access to the appropriate `/dev/rdisk` node can use hdiutil verbs such as fsid or pmap with it. Beware that information read from a raw device while a filesystem is mounted may not be consistent because the consistent data is stored in memory or in the filesystem's journal.

简单来说，就是 `rdiskx` 不用经过 block device 这一层抽象，也没有 buffer ，更接近底层，在操作时是直接对磁盘进行 IO 所以速度快。而 `diskx` 要经过一系列 buffer 和 block device 抽象的部分，这都是为了向 UNIX 用户提供统一的设备文件特性，而性能在其中有了损失。

虽然 `rdiskx` 这个文件因为接近底层，所以不是什么程序都能用（有点绕过文件系统的意思，manpage 也提到了），但是对于 `dd` 这种直接写磁盘映像的情况，可以尽管放心使用，获得 20x 到 40x 的速度提升。