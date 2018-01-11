---
layout: post
title: ZFS 与 LXC 与 GPU Passthrough，以及贵校超算队集群管理
date: '2018-01-11 23:10:00'
---

北京大学[超算竞赛队](http://pkusc.github.io)[1]有自己的实验/训练用机器，上面有很多的V100和P100，海量内存和不错的CPU，简直是<ruby>无所不能<rt>电老虎</rt></ruby>。所以机器闲着的时候，队员们可以上去跑一些自己的<ruby>计算任务<rt>TensorFlow</rt></ruby>。同时理所当然地，这台机器也成了实验室内外大家虎视眈眈的炼丹神器，机器上的用户越来越多。

每个UNIX/Linux用户都渴望自己拥有root权限，因为这样装新软件根本不用过脑子，直接用发行版打好的包就行。即使发行版没有打包，网上的无脑教程也往往假设用户拥有root权限。可惜的是，拥有root权限的用户往往意识不到自己同样负担有巨大的责任。比如我就听说过实验室里一台共用机器正跑着training，睡一觉醒来发现，前一天半夜有人把CUDA直接删了。这也就是同学们喜闻乐见的“丹炉炸了”的情形。这台机器后来root权限人手一个，丹炉也变成一天一炸、一天两炸、一天三炸。事实上，root权限在UNIX设计上仅供系统管理员使用，用的时候也是极其小心，仅仅是在安装关键依赖、修改系统配置时使用。至于用户自己要跑的程序，往往是手动从源码编译。可惜随着基于二进制的包管理成为Linux的主流，大家失去了“从源码编译安装”这一项古老而重要的技能。于是，用户要么自己拥有root权限，要么频繁求助系统管理员，两边都不舒服。

root权限泛滥的背后是隔离性的问题。比如高性能计算中至关重要的依赖——MPI：它有n个不同的实现，每个实现有m个不相兼容的版本，组合起来就是`nxm`种超凡享受，以至于出现一系列小脚本、小工具之类专用于切MPI库。再比如之前说到的CUDA，有人的神经网络框架要用老版本CUDA Toolkit，有人的要用新版本，于是最常见情况就是装了删、删了装，丹炉炸了又炸，好不热闹。类似的依赖项冲突的例子举不胜举。因此，一个公用的计算平台必须做到良好的隔离，这样每个人都可以随心所欲地用，用错了也都不会机器上的邻居们造成任何破坏。

今天，我们到了一台新机器 [1] 。这台机器是一台4U、8GPU插槽、20+磁盘位的机器，在接下来很长一段时间将成为我们的主力实验/比赛平台。结合上面的需求，我花了点时间好好配置了这台机器，希望能解决之前机器的麻烦。在这里把考虑和配置过程记在这里，防止之后参考。

# 存储与ZFS

一台4U机器有20多个盘位一点也不奇怪，这就是为什么我需要一个可以轻松扩展的存储系统。机器目前的存储设备有一块PCIe上的400G NVMe，以及SATA上的2T机械硬盘。因为超算应用有时候IO需求很猛，我们需要保证400G的NVMe能在必要时发挥全部性能并保证稳定性，所以它上面除了EFI和`/boot`之外，所有空间都分成一个ext4挂载在`/`。相比而言，2T的SATA盘主要是平时用来存些队员们的负责应用的实验数据和自己的<ruby>数据<rt>训练集</rt></ruby>。实验数据要跑时，会先拷贝到各自的用户目录下，所以这块HDD的任务就是作为数据存储池。之后很可能会买新的硬盘来提升池子的容量，或者加个镜像、或者组个RAID5什么的。所以我选择用ZFS管理这（些）HDD。

ZFS其实有不少“竞品”，hacky一点比如Linux LVM，full-blown一点比如btrfs。但是无论从稳定性上、声誉上还是功能上，这些竞品都不能和ZFS相比。我对ZFS抱有好感是从了解Mac OS X的文件系统开始。传说Apple一度想用ZFS作为Mac OS X的默认文件系统，来替代不断修修补补快要受不了的HFS。我知道后想“一个文件系统还能玩出花来？”，于是就去查ZFS的历史和功能，结果发现文件系统还真能玩出花来……后来在实验室的台机上用btrfs（Linux上的ZFS wanna-be），发现snapshot、subvolume、cow这些功能确实有用。再后来还开始用rsync.net [2] 、用ZFS作为LXC（本文的另一主角）的后端存储。相比btrfs一直以来的风波不断（不断有用户反映丢数据、发行版支持缓慢、被redhat放弃等等），ZFS的声誉一直很好，特别是在企业用户中。另外，我还特别喜欢ZFS的CLI Tool，对于现在这种简单需求，只需要几步就可以实现。

第一步创建一个storage pool。我们现在只有一块盘要加进，所以也不用考虑镜像/striping。

```
sudo zpool create yagami /dev/sda
```

我们就建好了一个叫做“yagami”的storage pool。

之后需要在这个pool上创建dataset，

```
sudo zfs create yagami/data
```

我们就创建好了一个叫做“data”的dataset。下面为“data”开启deduplication和compression，以节省硬盘空间

```
sudo zfs set dedup=on yagami/data
sudo zfs set compression=lz4 yagami/data
```

这样我们就有了存储池。它默认被挂在在`/yagami/data`。

事实上，LXC是可以用ZFS作为存储后端的，这也是我选择ZFS的重要原因。所以，也为LXC创建一个dataset。

```
sudo zfs create yagami/containers
sudo zfs set compression=lz4 yagami/containers
sudo zfs set dedup=on yagami/containers
```

去重（deduplication）对LXC是非常重要的，因为两个container，系统相关的文件很可能是一样的，去重效果会非常好。至此，ZFS就配置完毕。之后若要添加新的磁盘、或组镜像都非常简单，应该会省下很多事。

# Linux Container与隔离

要实现用户之间的隔离，方法有很多。简单粗暴的可以直接开虚拟机，更简单粗暴的有`chroot` jail之类，最简单粗暴的还有“无论如何都不给root权限，请本地编译安装”这个大招。隔离、性能损失与用户方便性这三点，一直是互斥的。KVM能实现良好的隔离，但是有相对较大性能损失，特别是GPU，很可能无法passthrough给用户。`chroot`没有额外开销，但是显得简陋，也几乎无法管理。Container在这种情况下成为了一个不错的选择。它事实上就是`chroot`、`cgroup`等东西包了一层，就像在同一个kernel上同时运行着多份userspace，因此几乎没有额外开销。但对每一个userspace来说，除了涉及kernel、driver的东西动不了，别的东西可以自由自在地控制，一个用户搞坏了也不会影响到另一个用户，真正实现“人人都是root用户”的理想。在我还自行host这个blog的时候，服务器上所有程序就都呆在各自的container里，配置起来可以放心大胆。LXC从几年前开始就非常好用，后来Canonical又推出了LXD，让LXC的管理更加简单，把我牢牢拉回Ubuntu坑。更好的是，LXD对GPU Passthrough的支持极好。Container用户可以获得和bare metal类似的GPU性能，这对我们的应用场景是必不可少的，因为我们的初衷毕竟是让用户可以互不影响地开心<ruby>跑实验<rt>炼丹</rt></ruby>。

机器上装的是Ubuntu 16.04，我们需要较新版本LXD上的一些新特性，因此需要从backports安装LXD。

```
sudo apt -t xenial-backports install lxd
sudo lxd init
```

`lxd init`时可以选择存储后端，此时选择ZFS，使用上一步创建的`yagami/containers`这个ZFS dataset。当问到要不要帮忙创建bridge时，选择不要。我们自己弄。`lxd init`创建的bridge会放在内网网段，container用NAT上网，明显不是我们想要的。我们希望每个container都能有一个外网地址，因此我们要创建一个bridge，并把目前的网卡attach上去。

```
sudo apt install bridge-utils
vim /etc/network/interfaces
# auto lo
# iface lo inet loopback
# 
# auto br0
# iface br0 inet dhcp
#   bridge_ports enp129s0f1
# 
# iface enp129s0f1 inet manual
```

这样一来，就把原来的网卡接到了`br0`这个设备下，`br0`通过DHCP获取IP。确认无误后后重启机器（其实有误导致断网也没关系，反正有IPMI）。

然后要让每个新开的container自动连上这个bridge

```
lxc network attach-profile br0 default eth0
```

LXC的配置就大功告成了，我们可以起一个container：

```
lxc launch ubuntu:xenial playground
lxc exec playground -- bash
```

每一个container都有自己的独立IP地址，运行自己的服务（比如sshd）。给每个用户分配一个container，他就在这台机器上有了自己的一片天地，可以随便折腾。

# GPU与GPU Passthrough

系统搭建到现在，就缺最重要的一环了——如何让container里面用上GPU？好在最近LXD提供了详尽的GPU Passthrough支持。其实硬件设备passthrough在虚拟机中一直是个头痛的问题，但在container中，因为container说到底还是物理机器上的kernel调度的一组权限受限的进程，物理机器的kernel和driver拥有对GPU 100%的访问。而用户程序使用GPU无非还是通过库，最后库去call那些个系统调用，这些系统调用最后落到物理机器的kernel去执行。显然物理机器的kernel执行这些访问GPU的系统调用是色宽 [3] 的，所以GPU Passthrough这件事情就一点也不头痛了。

LXD的GPU Passthrough支持简直不要太方便，加入所有GPU：

```
lxc config device add playground gpu gpu
```

加入指定编号的GPU：

```
lxc config device add playground gpu0 gpu id=0
```

还有更多控制方法。在添加GPU设备之后，container里就可以看到对应的GPU了。不过要真正用上它们，container里也需要安装必要的库或工具。比如`nvidia-smi`。需要注意的是，这些库必须和物理机器kernel里跑的GPU driver版本对应。因为container里装的库是在和物理机器的kernel配合工作。

配置完毕后，使用`nvidia-smi`查看在线的GPU。如果一切正常，就大功告成了。

[1]: 欢迎感兴趣的同学加入！请给我发邮件！
[2]: 一个很（回声无数次）棒的存储服务，后端是ZFS
[3]: 杭州话