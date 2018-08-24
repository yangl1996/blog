---
date: "2014-12-12T05:30:18Z"
title: 解决WNDR4300刷DD-WRT之后JFFS无可用空间的问题
---

刷上之后打开JFFS，发现居然没有可用空间。

一番搜索之后找到答案：http://www.dd-wrt.com/phpBB2/viewtopic.php?t=175215
<blockquote>1) mkfs.jffs2 -o /dev/mtdblock/3 -n -b -e 0x20000
2) mount -t jffs2 /dev/mtdblock/3 /jffs

Administration -&gt; JFFS2 support: Total / Free Size 102.75 MB / 100.12 MB

check also this: <a class="postlink" href="http://www.dd-wrt.com/phpBB2/viewtopic.php?t=171458&amp;highlight=wndr4300" target="_blank" rel="nofollow">http://www.dd-wrt.com/phpBB2/viewtopic.php?t=171458&amp;highlight=wndr4300</a>

Thanks to user habeIchVergessen</blockquote>