---
layout: post
title: 把 Cubieboard 4 作为 AP 使用
image: "/content/images/2016/07/IMG_0880.JPG"
date: '2016-07-21 15:32:23'
---

我的手头有两个空闲的 Cubieboard 4，这是个性能比较强的 ARM Mini-PC。为什么不叫它开发版？因为它的开源资源极差，可用的内核还停留在 3.4，而且编译起来极其麻烦，无法快乐地 hack 它。甚至社区还没有计划为它写 kernel patch。但是，它的外设很强，有 Gigabit Ethernet，802.11n@5GHz 和 Bluetooth 4.0 之类的。因此，它可以作为一个很好的 Mini Desktop PC 或 Mini Server。

最近我把实验室的网络重新部署了一下，需要有一个 AP 会比较方便，省得总是去连 Wireless PKU 还得占用连接数。这里的 AP 指的就是一个二层设备，相当于一个无线的交换机，和三层没有任何关系，不用分发 IP 也不用做 NAT。设备连上这个 AP 之后，会由上级的路由器负责 DHCP。

思路清楚之后，下面是配置过程。配置还是有一些坑的。使用的发行版是 Debian Wheezy（如果可选的话我是更喜欢用 CentOS）。

首先是启用 Cubieboard 4 上的 Wi-Fi 驱动。编辑 `/etc/modules`，加上

```
bcmdhd
```

这一行。

下面要把 `wlan0` 和 `eth0` 桥接。所谓桥接，就是相当于把这两个介面在二层连起来。首先安装 `bridge-utils`，然后重新编辑网络介面，修改 `/etc/network/interfaces`：

```
auto lo br0
allow-hotplug eth0

iface lo inet loopback

iface eth0 inet manual

iface br0 inet dhcp
    bridge_ports eth0
```

意思是，把 `eth0` 添加到 `br0`。然后 `eth0` 就不用配置 IP 了，取而代之，`br0` 用 DHCP 自动获取 IP（当然，如果你需要静态的，这里配成静态的当然也可以）。

这里不需要去管 `wlan0`，可以把与它相关的内容全部注释掉，因为一会儿 `hostapd` 会处理一切的。同样道理，也不需要手动把 `wlan0` 添加到 `br0`。

重启设备，可以看到 `br0` 工作正常了。到这里为止，网络相关的内容配置完毕，下面配置 AP。

首先，需要安装 `hostapd`。顾名思义，它就是用来 Host AP 的。经过实践，要用 v2.0 以上的才能让 WPA2 正常工作，所以考虑从源代码编译：

先安装依赖库：

```
apt-get update
apt-get upgrade
sudo apt-get install libnl-3-dev libnl-genl-3-dev libssl-dev
```

然后着手编译，先把源码 clone 下来，然后改一下配置，用 `libnl3.2` 并启用 802.11n 支持：

```
git clone git://w1.fi/srv/git/hostap.git
cd hostap/hostapd
cp defconfig .config
vim .config
```

把这些启用：

```
CONFIG_LIBNL32=y
CONFIG_IEEE80211N=y
CONFIG_DEBUG_FILE=y
CONFIG_HS20=y
```

然后编译安装即可。会安装在 `/usr/local/bin` 里。至此，`hostapd` 就安装好了。下面开始配置它：

创建 `/etc/hostapd/hostapd.conf`，下面是我的示例配置：

```
interface=wlan0
bridge=br0
driver=nl80211
country_code=CN
ssid=my-ssid 
hw_mode=a        # a for 5G, g for 2.4G
channel=165      # channel
ieee80211d=1
ieee80211n=1     # enable 802.11n support 
wpa=2
wpa_passphrase=my-password
wpa_key_mgmt=WPA-PSK
rsn_pairwise=CCMP
wpa_pairwise=TKIP
auth_algs=1
macaddr_acl=0
```

我这里用的是 5G，因为 5G 用户会少一点，信道阻塞情况好一些。

最后，`/usr/local/bin/hostapd -d /etc/hostapd/hostapd.conf`，就可以了。尝试连接一下，如果没什么问题，可以把 Debug 模式 `-d` 关掉，然后重新启动一下。也可以用 Deamon 模式启动，或者写进 `/etc/rc.local` 里开机启动。

最后测一下速度：在寝室，用 Cubieboard 4 做的软 AP，下行 30M，上行 47M，ping 4ms；用 WNDR4300，下行 58.4M，上行 104.05M，ping 5ms（没错，贵校校园网出口上行就是比下行还要快！！）。可以看出，比 WNDR4300 是要慢不少的。事实上，这个配置下，AP 工作在 20MHz 宽的频段里，理论速率是 65Mbps。而 WNDR4300 提供的的理论速率是妥妥的 450Mbps。

我尝试了不同方法，想让 Cubieboard 4 工作在 40MHz 的频道上，可惜它的驱动不支持 40MHz High Throughput，而且只支持比较低的 40MHz 频道，而我的 iPhone 不支持这些频道。权衡一下，还是放弃了。