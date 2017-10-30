---
layout: post
title: 'Veertu: macOS 下的 Native Virtualization'
image: "/content/images/2016/07/IMG_0882.JPG"
date: '2016-07-23 20:04:36'
---

虽然没有在笔记本上跑 Windows 的需求，但我还是会装一个虚拟机。事实上，虚拟化的好处远远不止“在 Mac 上跑 Windows 程序“那么简单。

就我而言，当要完成的工作会对系统产生未知的操作（比如执行一个需要 root 的脚本）时，我喜欢在一个新建的虚拟机里进行，然后事情做好连虚拟机一起删掉。这样可以保证 host 系统的安全，把系统维护成本降到最低。

我之前一直用 Parallels Desktop。这个虚拟机一度被称为 Mac 上的黑科技，因为它的性能确实非常好，而且有不少方便的功能，我试用半小时后马上就购买了，一年半以来也一直表现良好。

事实上，Parallels 提供了不少功能，可我都用不上：比如 Windows 应用程序窗口化在 Mac 中运行，再比如文件的自由拖放。相比这些，我更希望虚拟机能达到最大限度的隔离，最好是直接在沙箱中运行。当然之前这很难实现：为了达到更好的性能，虚拟机的调度应当陷入内核态，显然这对于一个沙箱内的程序来说是做不到的。好在 OS X Yosemite 开始，有了 Hypervisor Framework，它相当于提供了原生的虚拟化支持，任何用户态程序都可以通过调用 Hypervisor Framework 做虚拟机的调度，而不用亲自进入内核处理。

这样做有很多好处。首先，一个依托 Hypervisor Framework 实现的虚拟机不再需要改动内核。之前的虚拟机（VirtualBox，Parallels，VMWare Fusion）都会安装 kext 以做虚拟机的调度，如果这些 kext 出了安全问题，那问题是直接出来系统内核的。其次，这样的性能和能耗会更好。因为现在是由内核统一调度，统一管理能耗，所以虚拟机服从 host 的策略，也不会有各种 overhead。还有一个好处就是，沙箱化的虚拟机成为了可能，因为调用的全部是经过认证的原生 API。

Veertu 就是这样的一个虚拟机。它是目前唯二（另一个是 [xhyve](https://github.com/mist64/xhyve)）使用 Hypervisor Framework 的实现，也是唯一在 Mac App Store 可供下载的软件（另一个是个命令行工具）。

相比 Parallels，它有这些特点：

* 体积超小，只有 13.5MB
* 是沙箱程序
* 性能不劣于 Parallels
* 能耗优于 Parallels
* 在 macOS 的沙箱和 Hypervisor Framework 安全的前提下，虚拟机绝对安全
* 启动极快
* 不会改动系统
* 免费安装 Linux，若要安装 Windows 则价格低于 Parallels，而且貌似不是付费升级

这些优点大部分是拜 Hypervisor Framework 所赐，比如超小的体积：Veertu 本身就是个 GUI 壳，真正复杂的操作都是直接调用系统的 API。能耗，性能和安全性也都因为是原生的虚拟化。事实上，这样的虚拟机实现技术难度比 Parallels 这种小太多，我很好奇为什么没有类似的其他应用出现。

当然，由于它特殊的实现方式，一些在其他虚拟机上平常的功能 Veertu 是没有的，比如窗口化 Windows 程序，USB Passthrough，GPU Passthrough，富文本剪贴板之类的。这些对我来说都不成问题，如果有强需求的话，那 Veertu 就没有办法用了。

我已经开心地卸载了 Parallels 转用 Veertu。使用中还没有发现严重 bug 或性能问题。这个软件仍然在快速迭代中，新特性什么的都是可以期待一下的。