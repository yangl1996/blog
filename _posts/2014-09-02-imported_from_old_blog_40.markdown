---
layout: post
title: APRS Weather Box
date: '2014-09-02 08:06:20'
---

<em>说明：这篇文章原来放置在<a href="http://www.yangl1996.com" target="_blank">www.yangl1996.com</a>，但现在那里没地方放了，所以放到这儿来。英文介绍仍然在原处。</em>

<h3>概览</h3>

<p>APRS（Automatic Packet Reporting System，自动分组报告系统）是一个开放的AX.25数据网络，由鲍勃（WB4APR）创建。现在APRS被全世界的业余无线电爱好者广泛地使用，爱好者们通过这个网络交换各种信息。</p>

<p>本项目使用Arduino制作一个简易气象站，收集温湿度和气压信息。它每隔十分钟向APRS-IS服务器发送数据。 要自己搭建一个APRS Weather Box，你只需要一块Arduino板、一块W5100以太网盾板、一个DHT21温湿度传感器和一个BMP085气压传感器。 有关自制气象站和疑难解决的详细信息，敬请与我联系。 </p>

<p>代码由翁恺老师（BA5AG）和我编写。源代码放在<a href="https://github.com/yangl1996/aprswxbox">GitHub</a>上。</p>

<h3>准备</h3>

<ul>
<li>一台Mac或PC</li>
<li>Arduino IDE（版本1.00或更新）</li>
<li>一根USB线缆</li>
<li>一台路由器</li>
<li>电源</li>
<li>一块Arduino板</li>
<li>一块W5100以太网盾板</li>
<li>一个DHT21传感器</li>
<li>一个BMP085传感器</li>
</ul>
<h3>制作指南</h3>

<p>制作APRS Weather Box非常简单，只要连几根线就OK了。照着这篇指南完成所有步骤，用时大约15分钟。</p>

<p>首先，把以太网盾板插在Arduino上。把Arduino连上电脑，载入程序。之前先要根据个人信息，修改程序代码，让气象站发出的信标中带有你的呼号和位置。编辑 “callsign”、“passcode”和“location”。“passcode” 是一个独一无二的五位数字码，只有和对应的呼号一起使用才有效。它用来确认使用者的身份是业余无线电爱好者。如果你还没有这个验证码，可以向周围的HAM咨询。至于位置，你可以在Google Maps上查找，注意按照程序代码中的那种格式填写！如果需要（比如默认服务器宕机），也可以更改APRS-IS服务器地址。</p>

<p>之后，按照连线图所示连接两个传感器。注意确认连线的可靠性，以防传感器意外断开连接。</p>

<p>第三步。把气象站连上你的路由器。给Arduino上电。看看你的气象站“你的呼号-13”是不是正确出现在APRS网络上（比如在aprs.fi上搜索你的呼号）。如果出现了，那恭喜你，气象站成功搭建。如果出现问题，请试着参阅下面的故障排除指南。</p>

<h3>疑难排除</h3>
<p>如果你的气象站不工作或不正常，请参阅下面的详细解决步骤。</p>

<ol>
<li>在网线连好的情况下，把气象站连上电脑。</li>
<li>打开Arduino IDE中的串口监视器。</li>
<li>打开电源。</li>
<li>检查串口监视器中返回的信息。</li>
</ol><ul>
<li>如果信息与“连接错误”有关，请检查输入的APRS-IS服务器地址是否有误。</li>
<li>如果信息与“登录失败”有关，那可能是呼号、5位验证码输入错误。</li>
<li>如果信息是“DHT failure”或“BMP failure”可能是传感器连接错误。检查连接是否稳固，传感器是否坏掉了。</li>
<li>如果信息与“配置失败”，这代表气象站不能通过DHCP从你的路由器自动获取局域网IP地址。检查你的路由器的DHCP功能是否打开。</li>
</ul><p>尝试断电重启也可以解决偶然的程序卡死。如果这些都无法解决问题，那么请与我联系。我会尽量回复。</p>

<h3>参考资料</h3>

<p>连线图和源码放在<a href="https://github.com/yangl1996/aprswxbox">GitHub</a>上。BG5HSC也有一篇使用报告，放在他的<a href = "http://blog.sina.com.cn/s/blog_6ae7f76a0100zm4v.html">博客</a>上。</p>

<p>如果有任何问题，欢迎给我写邮件！我的电邮地址是i@yangl1996.com</p>