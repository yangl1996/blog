---
date: "2011-12-10T13:03:33Z"
title: 我的WordPress建站过程
---

我一直采用Google Sites建立网站，主要因为它没有任何门槛，而且有丰富的自定义选项。特别是在映射到自己的域名后，网站不需要翻墙就可以访问。但是在使用中，使用Google Sites的一些不便也暴露出来。比如发新页面需要翻墙才可实现（Google的大部分功能均被屏蔽），一些实用功能还不能提供。于是，我开始关注WordPress。经过三个双休日的安装后，我的WordPress网站终于建成了。
<ul>
	<li>免费的空间</li>
</ul>
找了一圈，参考了多人的资料后，最终确定一家德国的老牌免费空间提供商<a href="http://www.kilu.de">Kilu</a>。它的空间貌似比较稳定。申请过程不多讲，需要注意的是这个网站虽然支持中文，但是就像用Google翻译的一样，很难看懂，建议中、英文一起看。
<ul>
	<li>安装WordPress</li>
</ul>
WordPress是一个目前<em>最</em>优秀的博客（或者网站）平台！真的很好用。从<a href="http://cn.wordpress.org/">WordPress中国网站</a>下载官方的中文版本。修改其中的配置文件，使之符合你的空间的数据库帐户。具体修改方法，请自行搜索，此处不再一一介绍。接着，用FlashFTP等等软件将WordPress上传至空间。
<ul>
	<li>解决问题</li>
</ul>
这个时候基本上<em>都会</em>遇到一个问题（或者是两个）。以下是解决办法。
第一个问题，<strong>Error establishing database connection</strong>
一般来说，这个问题是由于在编辑好wp-config-sample后没有改名（改成wp-config）。只要把文件名改好后就OK了！当然，还有一种可能造成这一问题，就是在输入数据库信息的时候有误。这时只要改正后再次上传即可。
第二个，<strong>Cannot modify header information</strong>
怎么解决呢？先看看问题的由来。在建立WordPress站点的时候，会使用到wp-config-sample文件。编辑它才可以使站点链接到你的主机的数据库。但是，一般都是用记事本来编辑这一文件。记事本在编辑好文件另存为wp-config时，会在开头添加上一行BOM（字节顺序标志）。PHP是无法识别的（WordPress用PHP编写的）。
具体的解决办法是，在保存时ANSI格式（记事本保存时在“编码”里改变）。再次上传，你会发现问题已经解决了。
<ul>
	<li>完成它！</li>
</ul>
<blockquote>著名的五分钟安装</blockquote>
在浏览器中输入你的主机的域名，会出现WordPress的安装界面。输入你的网站标题（以后可以更改）等等信息，输入你以后管理网站的帐号密码，就完成了。
<ul>
	<li>更多的选项</li>
</ul>
登录你的网站，会出现控制板。在这里可以设定许多选项。在这里就不一一说明了。
<ul>
	<li>随处更新</li>
</ul>
WordPress的iOS平台版本十分方便，见截图


在你的设备上下载好WordPress的客户端之后，先别急着设置信息。
启用XML-RPC

然后设置你的帐户信息，享受吧！