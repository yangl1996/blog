---
date: "2017-04-09T18:27:09Z"
image: /content/images/2017/04/WWCC_240_litre_Recycling-_Green_waste_and_Garbage_bins-1.jpg
title: 在 Mac 上用 Openconnect 连接 Pulse Secure VPN
---

###### tl;dr

替换 Pulse Secure 的官方 Mac 客户端：
```bash
brew install openconnect
sudo openconnect --user 1400000000 https://vpn.pku.edu.cn --juniper
```

---

学校的 VPN 用的是 Pulse Secure（以前叫 Junos Pulse，后来分出去了）。这个 SSL VPN 在 Mac 上的客户端有几大麻烦的行为，导致我非常不想在电脑上装这东西，包括但不限于：

* 开机自启动，而且关不掉（其实可以关，就是把 LaunchAgent 里的 plist unload 掉，详见 Pulse Secure KB 里的[这篇文章](https://kb.pulsesecure.net/articles/Pulse_Secure_Article/KB26679)。但这样的话，每次要连 VPN 都得去重新 load，还是一样讨厌）
* 软件架构奇怪，tray 里的图标和 dock 里的貌似是分开的两个程序，关了一个另外一个毫无反应
* UI 烂到家（虽然比之前的 Junos Pulse 客户端已经改进了很多）

然后就找有没有什么东西可以取代它（当然是不抱希望地找），居然还真被我碰到了：[这篇博客](http://takuya-1st.hatenablog.jp/entry/2016/06/17/021346)提到可以用 Openconnect 取代它。作者也提到了 Pulse Secure 官方客户端的种种垃圾之处，看来广大用户朋友都用的不舒服w

Openconnect 是一个开源的 VPN 客户端，本来是为了解决 Cisco VPN 客户端在 Linux 上的诸多问题的，后来又加入了 Pulse Secure 支持（所以说 Pulse Secure 的 Linux 客户端估计也极烂）。项目网站很有趣，有一句

>  OpenConnect is not officially supported by, or associated in any way with, Cisco Systems, Juniper Networks or Pulse Secure. It just happens to interoperate with their equipment.

把锅甩的一干二净。可惜的是，我找到这篇文章的时候，Openconnect 还不能使用 Darwin 上 native 的 utun，而是必须用 [tuntaposx](http://tuntaposx.sourceforge.net)。这是一个 Mac 上的 TUN/TAP 实现。这就讨厌了，因为我对第三方 kext 的态度一向是能不装就不装。即使它开源，也担心影响 OS X 的稳定性（虽然本来就一般）。于是，用 Openconnect 替代原来客户端的计划暂时搁置了。

今天我翻 Safari 的 reading list，发现好久之前存的这篇文章，就决定去看看 Openconnect 是不是支持 utun了。结果还[真支持了](http://www.infradead.org/openconnect/building.html)！

> Newer versions of OpenConnect will use the utun device on OS X which does not require additional kernel modules to be installed.

赶紧装上试试。

```
sudo openconnect --user 1400000000 https://vpn.pku.edu.cn --juniper
```

效果完美。再也不用碰 Pulse Secure 自己的客户端了 > <