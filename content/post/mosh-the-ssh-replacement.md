---
date: "2017-04-23T13:28:20Z"
aliases:
    - /2017/04/23/mosh-the-ssh-replacement.html
image: /content/images/2017/04/DEC_VT100_terminal.jpg
title: Mosh：更好的 Remote Shell Access
---

SSH 作为一个远程访问协议，在提出时是为了解决 telnet 垃圾的安全性。在 22 年后的今天（SSH 在 1995 年由 Tatu Ylönen 提出）它已经成为了远程 shell 访问的事实标准。不过过了这么久，SSH 已经有一些与现在应用场景不适应的地方。2012 年，MIT 提出了 Mosh，希望解决这些问题，成为下一代的远程 shell 访问协议。之前在 hn 上看到过这个，但是那时候只有实验性质的客户端。最近发现它已经完全可用了。

### SSH 的问题

SSH 采用了 TCP，而 SSH 的问题一半就是 TCP 造成的。事实上，TCP 在 SSH 设计之初完全不算问题，反而是最好的选择。因为作为远程 shell 访问的应用，TCP 提供的特性（面向连接、可靠）简直就像量身定制，不直接用才奇怪。但是在现在，这种面向连接的设计却不够灵活了：TCP 的有状态连接使得设备在切换网络，或连接因其他原因断开的时候，就会丢失整个 session 无法恢复。比如我在 iPad 上用 SSH 客户端，中间去 Safari 查一个东西，查完回来发现连接被 kill 掉了（iOS 后台限制），就必须重新连，如果没有用 `tmux` 的话那之前的 session 就很难恢复了。同样的问题还可以发生在切换网络的时候：比如拿着手机从 WiFi 切换到蜂窝网络的时候。TCP 的这个问题也许可以用 MPTCP 解决，但是这个技术应用还太少（我只知道 Siri 用了）。

其他还有一些小问题，例如，大部分 ssh 客户端都是每敲一个字就发到对面，然后等着对面 echo，然后再显示在本地的 console 上。这在高延迟的网络下是非常痛苦的。尽管有些客户端会做 local echoing（例如 Windows 10 的 Token2Shell），但做的也一般。

### Mosh 的好处

Mosh 就是为了解决 SSH 在现在出现的几个问题而产生的。它提供的主要功能有：

1. 可断续的连接
2. 漫游（切换网络）
3. local echoing 增强高延迟时的用户感受

### Mosh 的实现

Mosh 首先放弃了 TCP，转而使用 UDP。UDP 完全是无状态的，“状态” 由应用层维护，尽管可以说增加了 overhead，但在现在硬件条件下可以无视了。用 UDP 的好处就是灵活。在实现上，Mosh 首先用现成的管道（如 SSH）连上服务器，然后交换密钥，之后启动服务器上的 Mosh server，这之后 SSH 就断开了。Mosh server 会开一个 UDP 端口，客户端再凭借刚交换的密钥去和 Mosh server 直接通信。Mosh 在这些 UDP 数据报上维护一个“连接”，保证数据顺序和完整性。如果客户端切换网络，也只需要和原来的 UDP 端口通信，告诉它现在的地址，这个“连接”就又恢复了。

为了保证高延迟时也能舒适的使用，在客户端这里可以开启 local echoing，把客户端的显示和数据的传输异步开来。用户输入字符后直接 echo 在屏幕上，而背后客户端先缓冲，由客户端判断何时发。例如，输命令的时候是即时的反馈，敲下回车后再做数据传输。

在安全性方面，首先，Mosh server 理论上是不会常驻的，它是由客户端通过 SSH initiate 的 Mosh session 触发的。所以说，只有可以通过 SSH 连上，才可以启动 Mosh server，才可以接受 Mosh 连接。因此，只要禁用 SSH 的密码登录（强烈推荐这么干），理论上服务器是安全的。而 Mosh 起来后，与客户端之间是 AES-128 加密。密钥交换在之前的 SSH session 中完成，因此也是不用担心的。

### 使用 Mosh

以 OpenSuSE 为例，

```bash
sudo zypper in mosh
```

然后如果开了防火墙、开了端口转发，那保证 60000-61000 UDP 可以通过（这是 Mosh 默认使用的端口）。然后就 OK 了。

Mosh 最佳的使用地点还是 iOS。桌面电脑不怎么出现网络切换，Android 的 SSH client 可以常驻后台，还是 iOS 最需要了。Mosh 以 GPLv3发布，另外对 App Store 有 waiver，允许在不破坏 GPL 其他限制的情况下在 App Store 上发布，因此 iOS 上的客户端也必须以 GPL 开源。现在的客户端有 [Blink](http://www.blink.sh)。这个 terminal emulator 做的特别 solid，性能甚至比 Panic 的 [Prompt 2](https://panic.com/prompt/) 还好。

在其他平台上，Mosh 也带来高延迟下更好的效果。它在几乎所有平台都有[对应的客户端](https://mosh.org/#getting)。我在一台东京的服务器上测试了一下（教育网访问延迟大约 300ms），可以明显发现本地的 echoing 和实际的字符传输是异步的，在本地的延迟感受明显消失了。