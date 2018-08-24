---
date: "2017-06-03T03:33:00Z"
aliases:
    - /2017/06/03/on-demand-ssh-tunnel-with-launchd.html
image: /content/images/2017/06/356656FF-9683-4407-8575-8CB602A3E2E2.jpg
title: 用 launchd 实现 on-demand 的 SSH tunnel
---

VNC 是 telecommute 的好方法。我经常 VNC 到实验室的机器和虚拟机上，使用装在上面的 MATLAB 和 Vivado。众所周知要把一个 VNC 服务器配的安全（证书认证 + 端到端加密）是个比较精细的活，所以最省力最安心的方法实际上是让 VNC server bind 到 localhost，然后用 SSH local forwarding 之类的方法连过去。所以一直以来我都是两步，第一步起 SSH local forwarding 第二步起 VNC client，像这样：
​
```bash
ssh -N -L 5903:localhost:5901 suse-519
open vnc://localhost:5903
```
​
每次都要先起 local forwarding，这么弄多了难免觉得麻烦。最近决定提升一下日常工作中的自动化水平，于是想把这个问题解决掉。最先考虑的是用 [SSH Tunnel Manager](https://www.tynsoe.org/v2/stm/) 之类的工具让 tunnel 一直在后台连着，但感觉这个方法有点不优美，而且为了这么个小需求装个 GUI App 有点没必要。然后突然想到[之前看到过](https://blog.yangl1996.com/use-openconnect-to-connect-to-pulse-secure-on-mac/) Pulse Secure 的客户端通过使用 launchd，在它的 GUI App 启动时自动起 daemon，这个情况和我的需求其实很像，于是觉得可以用 launchd 解决这个问题。
​
[Launchd](https://en.wikipedia.org/wiki/Launchd) 是 macOS 上的系统服务管理。它实际上是 Linux 社区中<ruby>风头正劲<rt>饱受争议</rt></ruby>的系统服务管理器兼 init 程序 <ruby>systemd<rt>shitstemd</rt></ruby> 的[灵感来源](https://news.ycombinator.com/item?id=7204093)。它提供很强大的系统服务管理，可以控制服务启动的条件、设置服务运行的环境、维持服务的运行。它的配置文件是普通的 `plist`，具体文件格式可以 `man launchd.plist`。一个 `plist` 如果被 `load` 了，launchd 就会按照配置文件的描述管理这个服务。
​
Launchd 很不错的一个功能是可以 launch on-demand socket。这个功能是从 UNIX 的 inetd 继承来的。Inetd 代替 server 进程 listen 端口，当端口被尝试连接时，inetd 启动对应的 server 进程，然后把 socket 的 `stdin`、`stdout`、`stderr` 交给 server 进程。这样一来 server 可以根本不管网络、socket 之类的，只需要在 `stdio` 上通信即可。Server 的控制逻辑（multithreading 等）和实际的通信分开，大大简化了 server 的设计，也方便统一管理 server 进程。这是一个很 UNIX，很优美的设计。我画了一张图说明一下 inetd 的工作方法：
​
![](/content/images/2017/06/inted-overview.png)
​
有了这个功能，要实现我的 on-demand SSH tunnel 功能就轻而易举了。让 launchd 监视 localhost 的 5903 端口，如果我的 VNC client 尝试连接它，就起一个 SSH。SSH 在实验室机器上起一个 `nc`，`nc` 与 localhost 5901（VNC 服务器所在的端口）建立 TCP 连接，并把这个连接的 socket 重定向到自己的 `stdio`。由于 `nc` 是 SSH 起的，SSH server 会把 `nc` 的 `stdio` 通过 SSH 连接往我这边的 SSH client 转发。我这边 SSH client 的 `stdio` 就接上了 `nc` 的 `stdio`，进而接上了 VNC 服务器。最后，利用 inetd 把 socket 重定向到 SSH client 的 `stdio`，这个 socket 也就和 VNC 服务器连接了。整个过程用语言描述比较复杂（其实没那么复杂），所以我还是画了张图：
​
![](/content/images/2017/06/ssh-tunnel-on-demand-whole-process.png)
​
Launchd 在提供 inetd 的功能时，做了一些<ruby>改动<rt>破坏</rt></ruby>。它要求被启动的程序用 `launch_activate_socket` 这个函数（参见 `man launch`）获得 socket 对应的 FD，然后用这些 FD 通信。这样的好处是，进程不必非得使用 `stdio` 与 socket 通信，而是可以使用预先想好的别的 FD，从而把 `stdio` 省出来做别的事（比如输出 log）。话说回来，这样的坏处也是显而易见的，那就是被启动的进程必须专门为 launchd 设计，在开头获取一次 FD，而不能直接使用通用的 `stdio`。好在 launchd 保留了对 inetd 的兼容。在 `plist` 中加上 `<inetdCompatibility>` 这个 key，launchd 就会按照 inetd 的方式工作，把 socket 重定向到 `stdio`。（如果 launchd 不提供这个选项，我们依然可以写一个 wrapper 来实现。Wrapper 做的事就是用 `launch_activate_socket` 获取 FD，然后启动服务进程，然后把 FD 重定向给服务进程的 `stdio`。）
​
最后，就可以写一个 `plist`。
​
```plist
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple Computer//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
    <dict>
        <key>Label</key>
        <string>com.yangl1996.sshtunnelforvnc</string>
        <key>ProgramArguments</key>
        <array>
            <string>/usr/bin/ssh</string>
            <string>-q</string>
            <string>-T</string>
            <string>suse-519</string>
            <string>nc localhost 5901</string>
        </array>
        <key>inetdCompatibility</key>
        <dict>
            <key>Wait</key>
            <false/>
        </dict>
        <key>Sockets</key>
        <dict>
            <key>Listeners</key>
            <dict>
                <key>SockServiceName</key>
                <string>5903</string>
            </dict>
        </dict>
        <key>SessionCreate</key>
        <true/>
    </dict>
</plist>
```
​
其中，
​
```plist
<key>inetdCompatibility</key>
<dict>
    <key>Wait</key>
    <false/>
</dict>
```
​
就是前面提到的兼容 inetd。`Wait` 设置成 `false` 表示 launchd 会不断接受新连接，每次 spawn 一个新服务进程去 handle。如果是 `true` 的话，就只允许同时有一个连接建立（同时只有一个服务进程运行）。
​
最后 `launchctl load` 这个服务，然后测试一下。可以发现，`open vnc://localhost:5903` 直接可用。然后本地出现了这些 socket：
​
```
tcp4       127.0.0.1.5903         127.0.0.1.64428        ESTABLISHED
tcp4       127.0.0.1.64428        127.0.0.1.5903         ESTABLISHED
tcp4       127.0.0.1.64426        127.0.0.1.5903         TIME_WAIT
tcp4       192.168.2.2.64429      222.29.98.192.22       ESTABLISHED
```
​
前两个是 launchd 和 VNC Client 之间的连接。第三个是 launchd 起了一个新连接，以允许 5903 继续接受连接（因为 `Wait=false`）。第四个是被 launchd 启起来的 SSH client。
​
然后看看各个进程打开的文件。
​
```
COMMAND FD  TYPE NAME
Screen  12u TCP  localhost:64428->localhost:5903 (ESTABLISHED)
ssh     0u  TCP  localhost:5903->localhost:64428 (ESTABLISHED)
ssh     1u  TCP  localhost:5903->localhost:64428 (ESTABLISHED)
ssh     2u  TCP  localhost:5903->localhost:64428 (ESTABLISHED)
```
​
可以看到，SSH 和 Screen Sharing（Mac 自带的 VNC 客户端）分别持有 `localhost:5903<->localhost:64428` 两端的 socket。SSH 这边，`localhost:5903->localhost:64428` 分别对应 FD 0、1、2，即 `stdin`、`stdout` 和 `stderr`。SSH 往 `stdout` 的所有输出，会进入这个 socket，然后到达 Screen Sharing。而 Screen Sharing 来的数据，则会到达 SSH 的 `stdin`。这和上面解释的原理是一致的。
​
关闭 VNC client 时，VNC session结束，TCP 连接被 VNC server 和 client 断开（`EOF`），`nc` 会退出。起 SSH 时，我们要求它在 remote machine 上运行的命令是 `nc`。现在 `nc` 运行完了，SSH session 也就结束了，我这边的 SSH client 进程也就结束了。至此，所有后台进程全部结束，SSH tunnel 非常干净地关闭了。
​
事实上，用 launchd 还可以实现很多<ruby>好玩<rt>有用</rt></ruby>的功能，还是蛮值得多尝试下的！

