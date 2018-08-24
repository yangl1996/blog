---
date: "2016-06-08T16:41:24Z"
aliases:
    - /2016/06/08/ti-xing-zi-ji-jing-chang-geng-xin-archlinux.html
title: 提醒自己经常更新Archlinux
---

刚刚在Cubieboard上装好Archlinux。知乎上有些人说滚动更新的发行版，太久不更新就容易“滚坏”。我可不希望哪次更新后就再也boot不了了。所以想找个方法保证更新频率。

首先想到的当然是搞一个cron job定时自动更新。但是这种做法是被强烈反对的，因为若在更新时产生了`.pacnew`和`.pacsave`文件，而又没人处理，是非常危险的。比如，某次更新把`/boot`目录下的`boot.scr`改了，那U-Boot就可能无法成功引导，也就进不了系统了。另外，重大更新前最好看看Arch的首页新闻，看看有没有坑。

反正，无人值守的自动更新是被strongly discouraged的。如果有需要，可以在`pacman -Syu`的时候多pass进一个`-w`表示只下载不更新，这样至少可以省掉用户等待下载的时间。但实际处理还是有人工介入比较好。

如果无法自动更新，那就只好想办法避免自己忘记了。首先确定上次更新是什么时候：这很简单，查pacman的log就行，在`/var/log/pacman.log`，里面有“starting full system upgrade”就表示尝试过更新。然后，用正则表达式提取出时间，再和现在时间算个差就行。

计算时间差可以用GNU date来做，把log里的时间日期转成UNIX time，然后和当前的UNIX time算差，再换算回小时或天就行。最后，如果天数大于6，就用红字打出警告。把这段脚本放进`.bashrc`里面就算大功告成，每次进shell都会提醒。这下不怕忘记更新了。

脚本放在[GitHub Gist](https://gist.github.com/yangl1996/44349cd3317be844e7dcfc60c0a05863)上。