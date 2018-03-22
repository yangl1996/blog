---
layout: post
title: 控制 Beaglebone Black 的 USB 电源输出
image: ''
date: '2016-10-22 15:55:57'
---

每次我在寝室想躺床上看看书，都会因为床上太暗而很难受，最终只好戴个头灯看。事实上我有一个宜家买来的 USB 小灯，亮度还不错，本来打算夹在床边当床头灯，只可惜它没有开关，所以用起来有些麻烦。

最近在寝室放了块 Beaglebone Black 用来把寝室和实验室用 VPN 连起来，刚才灵机一动，想到说不定可以通过控制 Beaglebone Black 的 USB 电源输出来开关灯，这样灯就可以一直插在 Beaglebone 上，然后用软件方法开关，可以衍生出各种玩法，岂不美哉！于是稍稍查询，发现确实可行，方法在这里记录如下。

Linux 内核本身并不提供强行给 USB 口通断电的功能，所以要采用别的方法：Beaglebone Black 的 USB Hub 由 GPIO3_13 口控制，只要改变这个口的输出，整个 USB Hub 供电也会相应改变。需要注意的是，这个方法在很多类似的板子上是行不通的：因为往往 USB 总线上接着各种板载设备，比如以太网，要是把整个 USB Hub 断了，可能这些设备也不工作。Beaglebone 的 USB 总线上什么都没接，于是可以放心地用这个方法控制 USB 口供电。

首先编译安装 `devmem2`，用于往 GPIO 写。

```sh
git clone https://github.com/VCTLabs/devmem2
make
make install
```

然后只需要往 GPIO 写就可以控制通断电了：

```
devmem2 0x47401c60 b 0x00  # Off
```
```
devmem2 0x47401c60 b 0x01 # On
```

如果要让驱动刷新 USB 口，就先 `unbind` 再 `bind`：

```
echo "usb1" > /sys/bus/usb/drivers/usb/unbind
echo "usb1" > /sys/bus/usb/drivers/usb/bind
```

在某次Linux Kernel更新之后，这个控制USB口电源的操作貌似不能可靠地完成了，所以我写了[一个小 script](https://gist.github.com/yangl1996/4e172a00be7c4fa9d308f9af6decfca7)在出错的时候自动重试。