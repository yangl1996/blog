---
layout: post
title: 自制Garmin手持机地图
date: '2011-12-25 07:47:00'
---

非常抱歉，图全部坏了。

<blockquote>这是我原来发在<a href="http://bbs.8264.com/thread-356056-1-1.html">bbs.8264.com</a>的帖子

&nbsp;</blockquote>
<ul>
	<li>前期准备</li>
</ul>
<ol>
	<li>性能好点儿的电脑（我的是amd的双核cpu，2g内存），至少别弄到一半卡牢了</li>
	<li>足够的耐心</li>
	<li>Global Mapper——用于自动生成矢量等高线数据，导出为MP格式</li>
	<li>cGpsmapper——用于把地图编译为garmin手持机支持的IMG格式</li>
	<li>MapEdit——用于把MP格式文件通过cGpsmapper转换成IMG文件</li>
	<li>img2gps——用于通过sendmap20把地图传至手持机（针对我的76cs一类的不带储存卡的手持机，带储存卡的可以无视）</li>
	<li>sendmap20——把IMG文件发送到手持机里</li>
	<li>迅雷（等多线程高速下载软件）——用于快速下载strm数据等文件，节约宝贵时间</li>
	<li>SRTM数据——美国佬的航天飞机扫描的地形数据，用于生成等高线（下载地址：<a href="http://srtm.datamirror.csdb.cn/index.jsp" target="_blank">http://srtm.datamirror.csdb.cn/index.jsp</a>）</li>
</ol>
<ul>
	<li>具体操作</li>
</ul>
1.打开global mapper，点“open your own data flies”图标，导入你的strm数据

2.等待一会儿，这是导入之后的样子

<a href="http://yangl1996-wordpress.stor.sinaapp.com/uploads/2011/12/1-01.jpg"><img class="alignnone size-large wp-image-84" title="1-01" src="http://yangl1996-wordpress.stor.sinaapp.com/uploads/2011/12/1-01-1024x819.jpg" alt="" width="584" height="467" /></a>

3.点flie菜单下的“generate countours”，弹出如下

<a href="http://yangl1996-wordpress.stor.sinaapp.com/uploads/2011/12/1-02.jpg"><img class="alignnone size-full wp-image-85" title="1-02" src="http://yangl1996-wordpress.stor.sinaapp.com/uploads/2011/12/1-02.jpg" alt="" /></a>

4.把等高距调为你需要的数值，这里为了速度，设为10米，再把平滑度调为0.00（最精细）

<a href="http://yangl1996-wordpress.stor.sinaapp.com/uploads/2011/12/1-03.jpg"><img class="alignnone size-full wp-image-86" title="1-03" src="http://yangl1996-wordpress.stor.sinaapp.com/uploads/2011/12/1-03.jpg" alt="" /></a>

<a href="http://yangl1996-wordpress.stor.sinaapp.com/uploads/2011/12/1-04.jpg"><img class="alignnone size-full wp-image-87" title="1-04" src="http://yangl1996-wordpress.stor.sinaapp.com/uploads/2011/12/1-04.jpg" alt="" /></a>

5.点ok，等一会儿（视数据复杂程度和范围大小而定，也可以选定生成等高线的范围，自己摸索吧，这里为了快点儿，先选一小块）

6.global mapper这个时候会卡一会儿，再等一会儿

7.欣赏几秒钟，到此，等高线生成工作完成
8.点flie菜单——export vector data——export polish mp 什么什么的
9.直接点ok（其实这里也可以选择输出数据的范围，但是也请自己摸索）
10.现在关闭global mapper，用mapedit打开刚刚导出的那个mp格式数据

<a href="http://yangl1996-wordpress.stor.sinaapp.com/uploads/2011/12/1-05.jpg"><img class="alignnone size-full wp-image-88" title="1-05" src="http://yangl1996-wordpress.stor.sinaapp.com/uploads/2011/12/1-05.jpg" alt="" /></a>

11.点flie菜单——export——garmin img 什么的12.找到要保存的地方，点“保存”，然后出现如下对话框，输入你的cgpsmapper的路径

<a href="http://yangl1996-wordpress.stor.sinaapp.com/uploads/2011/12/1-06.jpg"><img class="alignnone size-full wp-image-89" title="1-06" src="http://yangl1996-wordpress.stor.sinaapp.com/uploads/2011/12/1-06.jpg" alt="" /></a>

13.点run，然后过一会儿会出现提示，告诉你文件编译成功（希望是），然后关了这些，已经离成功很近了
——注：到这儿位置，使用带有储存卡的手持机的就可以把这个文件放到储存卡的目录里，具体我也不会，因为我的手持机是不带储存卡的，可以去搜索一下——
14.拿出你的garmin地图机，连上电脑（你应该先装好usb驱动，在Garmin官网上有）
15.打开img2gps，点load folder按钮，载入你要的img文件，然后应该是这样的界面：

<a href="http://yangl1996-wordpress.stor.sinaapp.com/uploads/2011/12/1-07.jpg"><img class="alignnone size-full wp-image-90" title="1-07" src="http://yangl1996-wordpress.stor.sinaapp.com/uploads/2011/12/1-07.jpg" alt="" /></a>

选中这（几）个文件，点upload to gps，然后（应该）会弹出一个黑框框，再等一会儿，等进度变为100%时，地图就传好了！！！
<div style="text-align: left;">
<ul>
	<li>其他</li>
</ul>
</div>
<div style="text-align: left;">
<ol>
	<li>用到的大部分软件都可以从<a href="http://cid-b9fb0bb88e13c9d4.skydrive.live.com/browse.aspx/Software" target="_blank">http://cid-b9fb0bb88e13c9d4.skydrive.live.com/browse.aspx/Software</a>下载</li>
	<li>这样做出来的地图没有地名数据（主要是因为国内的矢量地名、道路等数据实在缺乏），只能看地形，但是也可以在没有道路的情况下规划路线，别太高估它</li>
	<li>可以看看磨房赵版的经典帖子<a href="http://www.doyouhike.net/forum/comm_nav/144782,0,0,1.html" target="_blank">http://www.doyouhike.net/forum/comm_nav/144782,0,0,1.html</a>，讲的是使用global mapper做等高线地形图的方法，写的肯定比我好，建议借鉴</li>
	<li>台湾的mobile01上有关于做带有道路数据的矢量地图的方法：<a href="http://www.mobile01.com/topicdetail.php?f=130&amp;t=59066" target="_blank">http://www.mobile01.com/topicdetail.php?f=130&amp;t=59066</a>，有这方面需要的也可以看看（被墙）</li>
</ol>
</div>