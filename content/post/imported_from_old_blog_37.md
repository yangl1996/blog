---
date: "2014-08-30T07:23:42Z"
aliases:
    - /2014/08/30/imported_from_old_blog_37.html
title: 做了一个Status Page
---

最近一直折腾L2TP模式虚拟专用网，一直搞不定，所以做一点别的玩玩。

可能有TX看到过<a href="http://status.github.com" target="_blank">status.github.com</a>，这个是GitHub用来通报网站可用性的页面，他的各种服务如果出现问题，都会在这个页面有所体现。

我想自己也提供一个类似页面。简单搜索之后在V2EX论坛上找到些信息，<a href="http://www.uptimerobot.com" target="_blank">UptimeRobot</a>。提供网站监控的服务不少，但是这一家有比较实用的API，所以可以根据这个来开发一个展示网站可用性的页面。这个工作已经有人做好了：<a href="https://github.com/digibart/upscuits" target="_blank">Upscuits</a>。这个就是调用了Uptime Robot的API，然后进行了可视化。整个Web App就是个静态页面，所以找地方放非常容易，我是放在GitHub Pages上：<a href="http://status.yangl1996.com" target="_blank">status.yangl1996.com</a>，目前效果很棒。

整个设置过程也很简单，只需要稍稍修改配置文件，输入自己网站对应的监控API就好。具体的安装过程在Upscuits的repo的readme里面有。大概几分钟就可以全部搞定。

如果你自己也想做一个这样的status page，可以直接fork我的<a href="https://github.com/yangl1996/status" target="_blank">这个repo</a>，然后修改一下gh-pages分支下的/js/config.js里面的API Key（先注册Uptime Robot并获得API Key），status page就会在Your-github-username.github.io/status下运行啦。