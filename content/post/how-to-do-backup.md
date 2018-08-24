---
date: "2017-05-16T23:22:00Z"
aliases:
    - /2017/05/16/how-to-do-backup.html
image: https://blog.yangl1996.com/content/images/2017/05/Osaki_Station_-Rinkai_Line_and_Shonan-Shinjuku_Line-.jpg
title: 讲一讲备份
---

前几天的 [WannaCry](https://zh.wikipedia.org/wiki/WannaCry) 勒索软件给很多 PC 用户都留下了深刻印象，在国内重灾区是教育网。对于没有备份习惯的用户，就是要么交钱，要么放弃数据的选择。但对于有良好备份习惯的用户，被攻击的后果顶多也就是重装下系统。这次攻击（hopefully）给大部分普通用户形象地展示了备份的重要性。事实上，备份的好处远不止防备这种 ransomware。从计算机被盗、手残误删，到硬件故障乃至自然灾害，良好的备份习惯都是有效的保险手段。

### Rule of Thumb: The 3-2-1 Principle

什么算是良好的备份习惯？United States Computer Emergency Readiness Team 在 2012 年推广了一个[指导原则](https://www.us-cert.gov/security-publications/data-backup-options)（本身不是 US-CERT 写的，是一本 O'Reilly 的书上提出的），简单来说就是 “3-2-1”：

> To increase your chances of recovering lost or corrupted data, follow the 3-2-1 rule:
>
> 3 – Keep 3 copies of any important file: 1 primary and 2 backups.
>
> 2 – Keep the files on 2 different media types to protect against different types of hazards.
>
> 1 – Store 1 copy offsite (e.g., outside your home or business facility).

所以说，“3-2-1” 指的是：

* 数据总共要有三份拷贝（一份原始数据，两份备份）
* 使用两种存储介质
* 要有一份异地备份

第一条是确保有足够多的冗余，防备极小概率事件。第二条中两种介质预防不同的风险，比方说 CD 可能被划伤、磁带可能被强磁场损坏，但这两个事故一般不会同时发生。第三条防止物理上毁灭性的损坏，比如自然灾害。如果可以做到这三条，那数据的安全就有了一定的保证。

事实上，仅有良好的备份习惯是不够的。更重要的是要确保辛辛苦苦维护的备份在真正事故时可以派上用场。一份备份可能因为各种莫名其妙的原因就失效了。比如说 data rot（硬件中存放的数据在离线状态下缓慢损坏），或者再也没有软件可以打开备份，等等。今年 2 月 1 号 GitLab 因为所有备份方法[全部失效](https://about.gitlab.com/2017/02/01/gitlab-dot-com-database-incident/)，最后丢了 6 个小时的数据。所以说定期试用备份是[非常重要](http://checkyourbackups.work)的。

### 简单的实现方法

Mac 用户很早以来就有 Time Machine 可以用。Time Machine 会对整个磁盘做增量备份，并保存所有文件的历史记录，这样就算突然想找回一年前删掉的文件也不费吹灰之力。所以说，__强烈建议__各位 Mac 用户买一个 2T 或 4T 的 USB 3.0 硬盘__专门__用于 Time Machine 备份。我的 Time Machine 存了将近 4 年的全磁盘的文件历史，到现在也才用了 1.58TB，实际上是很紧凑的。之前我见过 Mac 硬盘坏掉、主板突然坏掉开不开，然后担心数据找不回来急得要死的，如果平时用 Time Machine 的话这些问题都完全不存在。

Windows 从 Windows 10 开始加入了“文件历史记录”功能。之前 Windows 的系统还原功能比较傻，所以很多用户选择不启用。“文件历史记录”就好多了，和 Time Machine 的功能几乎一样，__强烈建议__ Windows 10 用户买个 USB 硬盘然后启用。

Linux 下的备份花样就很多了。支持 Snapshot 的文件系统（ZFS、BtrFS 之类的）可以直接定期做全文件系统的 Snapshot。这里要安利一下 OpenSUSE，安装时默认 `/` 用 BtrFS，`/home` 用 XFS，然后每次修改配置、安装包前后自动对 `/` 做 Snapshot，并支持从 GRUB 直接 boot 进这些 Snapshot 并从中恢复系统配置。我用上过一次，方便的不行。除了依靠文件系统，还可以用 `rsync` 同步到远端的文件系统，或者用 `borg` 做自动增量备份。`borg` 这个工具是我刚刚发现的，非常不错。它把文件无差别切成 `chunk`，然后在 `chunk` 之间排重，效率很高。

### 异地备份

前面提到的都是简单易行的备份方法，但基本是本地的备份，依靠一块 USB 硬盘。但如果 USB 硬盘和电脑一起砸地上坏了怎么办？所以对于真正要紧的数据，需要异地备份，也就是 “3-2-1” 原则中的 “1”。

异地备份的方法非常多，简单的比如上传到网盘、使用各大服务商的“云”服务之类。比方说，iOS、Mac OS 用户可以扩大 iCloud 存储空间，然后把所有照片、音乐、日常工作文件都放到 iCloud Photo Library、iCloud Music Library 和 iCloud Drive 上，再给 Apple ID 用上 [2FA](https://support.apple.com/en-us/HT204915)（__很重要__），这些数据基本就万无一失了。

比这些复杂一些，可以自己在异地维护一个存储。例如，在实验室放一台小机器，装几块硬盘，然后电脑自动往这上面备份。再比如，租一个服务器，然后往上 `rsync`。

也有专门的服务提供这样的异地备份存储。AWS 的 Glacier 提供低至 $0.004 每 GB 每月的半离线存储，假设备份大小是 500GB 的话，每个月开销是在 ¥15 以下的，但却享受了 AWS 的服务水平和他内部的异地容灾。Google 也有类似的服务叫 Nearline 和 Codeline。还可以用 S3 做热备份，然后定期自动 dump 到 Glacier，等等各种玩法。另外，还有 Tarsnap、rsync.net、Backblaze 之类小一些的公司，也提供类似的备份存储。其中，rsync.net 是从 2001 年就开始服务的，历史比 AWS 要久的多。

### 那么问题来了：做到这些，我是不是就不怕 ransomware 了？

不能这么想。设想这种情况：你把硬盘插电脑上让他自动备份，然后睡觉去了。醒来发现被攻击，然后全硬盘、连带移动硬盘全被加密。由于备份工具逻辑失误，加密后的内容被同步到了你的异地备份，所有数据全部完蛋。类似情况还有很多，比如别人非法访问你的电脑，然后删光所有数据，顺便帮你登上服务器把异地备份也删精光。所以说，不能指望任何防御措施可以完美地防范威胁。所以说有良好的使用习惯也是很重要的，比如定期给异地备份打 Snapshot 并设置为 immutable、绝对不要无视 Windows 和 Mac 的软件证书警告、定期关注相关新闻及时装补丁之类的。
