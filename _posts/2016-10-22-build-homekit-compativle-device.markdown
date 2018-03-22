---
layout: post
title: 自制 Homekit 和 Alexa “智能家居”设备
image: "/content/images/2016/10/IMG_1158.jpg"
date: '2016-10-22 19:28:17'
---

__注：__ 本文最初写于2016年10月22日，那次做的是HomeKit的设备，后来这个BeagleBone被用来干其他的事情了。今天（2018年3月22日）比较有空，决定把这个声控小灯恢复，这次用了Amazon Alexa，使用暑假买的Amazon Echo Dot控制，过程添加在文末。

---

整个尝试起源于我想躺在床上看纸质轻小说：开学前在 Kindle 里放了 6 本，然后又跑到朝阳门买了 5 本纸质的，结果 Kindle 里的两周就看完了，纸质的到现在才看了 1/6 本。问题被归结到床上照明不好，看纸质书比较累，于是我决定在床边装个 24 小时可以亮的灯。

现有的设备有床底下的蓄电池、Beaglebone、一个宜家的 USB 小灯（没开关）。我最初的想法是把小灯接到 Beaglebone 上，Beaglebone 本身就接着蓄电池以保证 24 小时供电（用于寝室和实验室之间的 VPN），然后只要控制 Beaglebone 的 USB 口输出就控制了灯。这个事情马上就做完了，记录在[这篇文章](https://blog.yangl1996.com/control-beaglebone-black-usb-power-output/)。

然后考虑如何加强一下易用性：每次都 ssh 上去输行命令开关灯显然太麻烦了，一个成熟的方法是写一个简单的网页，然后手机访问它来控制，但是操作还是有点慢。要是能整合进 Apple Homekit，让手机、手表都能直接开关，还能用 Siri 控制，那就爽了。

Apple Homekit，很可惜，不对个人开发者开放，只开放给拥有 MFi 认证的厂商。好在 Homekit 出来有一段时间了，该做的反向工程都有人做完了，并把结果放了出来。GitHub 上就有几个类似的项目，比如我采用的 [HAP-NodeJS](https://github.com/KhaosT/HAP-NodeJS)：它是一个 Homekit Accessory Server，可以用于创建任何种类的 Homekit Accessory。下面是我使用它自制 Homekit Compatible 床头灯的过程。值得一提的是这个 project 的依赖管理做的不是特别好，直接 `npm install` 之后有相当概率出各种错，这里的过程只是我在 Archlinux 上把它跑起来并实用的过程。

### 安装 HAP-NodeJS

这是个 node.js 项目，但是仅仅 `npm install` 完全无法解决其依赖问题。

首先安装 `avahi` 并启动之。

```
pacman -S avahi
systemctl restart dbus
systemctl daemon-reload
systemctl start avahi
systemctl enable avahi 
```

然后安装编译 node.js 包需要的依赖

```
pacman -S python2
npm install -g node-gyp
```

然后把源码 clone 下载，并尝试安装依赖

```
git clone git@github.com:KhaosT/HAP-NodeJS.git
npm install
```

我在 `npm install` 完之后貌似没有装好所有依赖，还补充了这三个

```
npm install mdns
npm install libxmljs
npm install ed25519
```

全装好以后，可以运行了。运行的时候会报一些 warning，查了下 export 某个环境变量可以 supress 掉，所以我暂时没管。

```
node Core.js
```

这个时候打开 Home App 并尝试配对，已经可以看到一堆设备了。

### 编写设备脚本

我把床头灯的脚本放到 [GitHub](https://github.com/yangl1996/hap-nodejs-accessories/tree/master) 上了。

HAP-NodeJS 的设备脚本非常容易写，只要有个例子，稍微改一下就可以用了。在 HAP-NodeJS 提供的灯的脚本基础上，我做了这些修改（变量名都懒得改了，还是 “FAKE”）：

```
var cmd_on = "sudo devmem2 0x47401c60 b 0x01";
var cmd_off = "sudo devmem2 0x47401c60 b 0x00";
setPowerOn: function(on) { 
    if (on) {
        exec(cmd_on, function(error, stdout, stderr) {
            console.log("Turning on");
            FAKE_LIGHT.powerOn = true;
        });
    }
    else {
        exec(cmd_off, function(error, stdout, stderr) {
            console.log("Turning off");
            FAKE_LIGHT.powerOn = false;
        });
    }
},
```

这段定义了当灯被要求开启时，server 该做什么（往 GPIO 口写从而给 USB 口通电）。顺便注释掉了后面调亮度的函数，因为没法调。同时也把 expose 调亮度功能给外界的那段删了。

```
light.username = "1A:2B:3C:4D:5E:6F";
light.pincode = "123-45-678";

light
  .getService(Service.AccessoryInformation)
  .setCharacteristic(Characteristic.Manufacturer, "Lei Yang")
  .setCharacteristic(Characteristic.Model, "Rev-1")
  .setCharacteristic(Characteristic.SerialNumber, "A1S2NASF88EW");
```

这里设置灯的 PIN code，配对的时候要输入的八位数码。还有一些简单的信息。

```
light
  .addService(Service.Lightbulb, "Bed Light") // services exposed to the user should have "names" like "Fake Light" for us
  .getCharacteristic(Characteristic.On)
  .on('set', function(value, callback) {
    FAKE_LIGHT.setPowerOn(value);
    callback(); // Our fake Light is synchronous - this value has been successfully set
  });
```

这段实际上创建了一个灯设备（抽象为 Service），并命名为 Bed Light。当它捕获到 iPhone 发来的开关信号时，就调用之前定义的
`setPowerOn` 函数。很合理。

```
light
  .getService(Service.Lightbulb)
  .getCharacteristic(Characteristic.On)
  .on('get', function(callback) {
    var err = null; 
    var check_cmd = "sudo devmem2 0x47401c60";
    exec(check_cmd, function(error, stdout, stderr) {
        if (stdout.indexOf("0x117916697") > -1) { 
            console.log("Are we on? Yes.");
            callback(err, true);
        }
        else {
            console.log("Are we on? No.");
            callback(err, false);
        }
    });
});
```

这段用于检查并返回灯的状态。本来是直接无脑返回 cache 的状态，但这样健壮性未免太差了。所以这里改成用 `devmem2` 检查 GPIO 口状态（USB 有没有通电）并返回。

至此，这个设备就完全可用了。现在只需要 `node Core.js` 并配对就可以啦！从 Siri 和 Home App 操作都没问题，也可以把 iPad 放在寝室做 Remote Access Server 提供远程控制。

---

### Amazon Alexa

原来的HomeKit小灯因为BeagleBone被我拆走干别的事情而停用了一段时间。今天把BeagleBone重新刷了OS，决定把这个声控小灯恢复。之前HomeKit小灯的设置过程比较痛苦，最主要的原因就是它用了node.js。我的BeagleBone Black只有512M内存，而一个`npm install`是可以轻松用掉2G的（`npm`实在是太垃圾了，真不知道JS community怎么想的）。正好暑假买了一个Echo Dot，正愁没地方用，于是决定做一个和Alexa兼容的。

Echo出来这么久，早就有人实现了[和它兼容的smart home device bridge](https://github.com/toddmedema/echo)。不过这个是个Python 2的程序，我正准备帮他改成Python 3的。

搭建过程非常方便，首先确保有Python 2、pip，然后Echo和板子在同一个广播域内。然后clone[那个project](https://github.com/toddmedema/echo)并用pip安装所有依赖。Project里提供了一个example，第30行就是用来处理Echo发来的控制命令的。只要把它改成开关灯的code就可以了。另外，第27行设定了这个设备使用的名字和端口。这个名字就是之后向Echo说话时指代这个设备用的，可以改掉。

配对过程也非常方便。和Echo说“discover my devices”，大概10秒就配对成功啦！