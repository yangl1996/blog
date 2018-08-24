---
date: "2011-12-11T05:05:43Z"
aliases:
    - /2011/12/11/imported_from_old_blog_21.html
title: Arduino+Ethernet shield：入门，服务器，微博
---

<blockquote>让你的Arduino上网</blockquote>
<ul>
	<li>引言：Ethernet和Ethernet Shield</li>
</ul>
Ethernet其实就是我们熟悉的以太网。这是我们使用最广泛的局域网系统。Ethernet板可以让Arduino具备连接到以太网的能力。这就是Ethernet Shield：

<a href="http://yangl1996-wordpress.stor.sinaapp.com/uploads/2011/12/20111217-094336.jpg"><img class="alignnone size-full wp-image-48" title="Ethernet Shield" src="http://yangl1996-wordpress.stor.sinaapp.com/uploads/2011/12/20111217-094336.jpg" alt="" /></a>
<ul>
	<li>准备清单</li>
</ul>
要尝试下面写的试验，你需要准备
<strong>硬件——准备这些，你至少可以完成第一个实验：</strong>
<ol>
	<li>Arduino Duemilanove 或 UNO</li>
	<li>Ethernet Shield</li>
	<li>一条USB线</li>
	<li>一条以太网线</li>
	<li>路由器</li>
</ol>
<strong>软件<strong>——准备这些，你至少可以完成第一个实验</strong>：</strong>
<ol>
	<li>Arduino IDE（集成开发环境）</li>
	<li>Ethernet库（Arduino IDE中一般直接就有）</li>
</ol>
<strong>如果准备这些，你可以完成更多内容：</strong>
<ol>
	<li>一个新浪微博帐号</li>
</ol>
<ul>
	<li>先把硬件搞定</li>
</ul>
板子的连接超级简单，把Ethernet Shield直接插在Arduino上（注意保护针脚）

<a href="http://yangl1996-wordpress.stor.sinaapp.com/uploads/2011/12/20111217-100043.jpg"><img class="alignnone size-full wp-image-49" title="Ethernet Shield的连接" src="http://yangl1996-wordpress.stor.sinaapp.com/uploads/2011/12/20111217-100043.jpg" alt="" /></a>

<strong>需要注意的是，</strong>我在连接完后发现，Arduino的USB接口外壳是金属的，为了防止不小心把上层的Ethernet Shield短路，我在两块板子的接触部位垫了一张纸。图中的白色部分就是这张纸。

<a href="http://yangl1996-wordpress.stor.sinaapp.com/uploads/2011/12/20111217-100502.jpg"><img class="alignnone size-full wp-image-50" title="谨防短路" src="http://yangl1996-wordpress.stor.sinaapp.com/uploads/2011/12/20111217-100502.jpg" alt="" /></a>
硬件连接就这样完成了！
<ul>
	<li>软件:步骤一，最基础的效果</li>
</ul>
在Arduino IDE安装目录下找到这个文件:<strong>WebServer.pde</strong>

<a href="http://yangl1996-wordpress.stor.sinaapp.com/uploads/2011/12/2.jpg"><img class="alignnone size-full wp-image-51" title="文件路径" src="http://yangl1996-wordpress.stor.sinaapp.com/uploads/2011/12/2.jpg" alt="" /></a>
打开它。
现在需要编辑一下这个文件。这里给出了代码。
<code>/*
* Web Server
*
* A simple web server that shows the value of the analog input pins.
*/

#include

byte mac[] = { 0xDE, 0xAD, 0xBE, 0xEF, 0xFE, 0xED };
byte ip[] = { 10, 0, 0, 177 };

Server server(80);

void setup()
{
Ethernet.begin(mac, ip);
server.begin();
}

void loop()
{
Client client = server.available();
if (client) {
// an http request ends with a blank line
boolean current_line_is_blank = true;
while (client.connected()) {
if (client.available()) {
char c = client.read();
// if we've gotten to the end of the line (received a newline
// character) and the line is blank, the http request has ended,
// so we can send a reply
if (c == 'n' &amp;&amp; current_line_is_blank) {
// send a standard http response header
client.println("HTTP/1.1 200 OK");
client.println("Content-Type: text/html");
client.println();

// output the value of each analog input pin
for (int i = 0; i &lt; 6; i++) {
client.print("analog input ");
client.print(i);
client.print(" is ");
client.print(analogRead(i));
client.println("br /");
}
break;
}
if (c == 'n') {
// we're starting a new line
current_line_is_blank = true;
} else if (c != 'r') {
// we've gotten a character on the current line
current_line_is_blank = false;
}
}
}
// give the web browser time to receive the data
delay(1);
client.stop();
}
}</code>
把byte IP改成你需要的地址。一般改为192.168.1.177即可。
接着，上传文件到Arduino。<strong>需要注意的是，</strong>我一开始在传时一直弄不好，后来才发现是串口号忘记选了。在上传程序的时候一定要记得选择正确的，对应你的Arduino板的串口。
<ul>
	<li>测试！</li>
</ul>
用以太网线把板子连接到路由器的LAN（局域网）口。
在浏览器中输入你刚刚输入的IP地址，比如192.168.1.177
如果一切正常，你应该会看到这样的内容

<a href="http://yangl1996-wordpress.stor.sinaapp.com/uploads/2011/12/3-300x94.jpg"><img class="alignnone size-full wp-image-52" title="效果" src="http://yangl1996-wordpress.stor.sinaapp.com/uploads/2011/12/3-300x94.jpg" alt="" /></a>

这时Arduino Analog口当前的读数。如果你看到了，那么恭喜你，你的Arduino已经连接到你家的局域网了。
<ul>
	<li>在任何地方访问！</li>
</ul>
现在，Arduino只是连接在你家的局域网，只有局域网内可以访问。现在，我们让它可以在任何地方访问。这需要你家的路由器支持<em>虚拟服务器</em>功能（一般的路由器都支持）。
首先，登录你的路由器的管理平台。找到<em>虚拟服务器</em>功能。虚拟服务器使得英特网上的每个用户都可以通过你的路由器，读取特定WAN口上的内容，比如访问你的Arduino服务器。

<a href="http://yangl1996-wordpress.stor.sinaapp.com/uploads/2011/12/20111217-154834.jpg"><img class="alignnone size-full wp-image-53" title="虚拟服务器" src="http://yangl1996-wordpress.stor.sinaapp.com/uploads/2011/12/20111217-154834.jpg" alt="" /></a>
添加一条新内容。如果你使用的是192.168.1.177，那么就像这样

<a href="http://yangl1996-wordpress.stor.sinaapp.com/uploads/2011/12/20111217-155202.jpg"><img class="alignnone size-medium wp-image-54" title="设置虚拟服务器" src="http://yangl1996-wordpress.stor.sinaapp.com/uploads/2011/12/20111217-155202-300x225.jpg" alt="" width="300" height="225" /></a>
好了！
现在，找到你家现在的IP地址（这个IP地址是你的互联网服务提供商给你的，你可以在你路由器的状态信息中找到）。
在浏览器中输入它，接着，你就会看到刚才出现的页面。不同的是，现在在任何地方都可以访问它。你可以把这个地址告诉你的亲戚朋友，让他们试一试。
<ul>
	<li>软件:步骤二，加点内容</li>
</ul>
我们可以往小服务器里加一些内容，让它显示我们自己的东西，比如一段话。
先从加入文字开始。先看一看程序文件，不难发现它显示文字的是这一段
<code>// output the value of each analog input pin
for (int i = 0; i &lt; 6; i++) {
client.print("analog input ");
client.print(i);
client.print(" is ");
client.print(analogRead(i));
client.println("br /");
}
break;
}</code>
用了一个循环，来显示模拟口1-6的读数。
这里的<em>client.print</em>用来输出文字，我们就用这个函数来写自己的文字。
<em>client.println</em>用来写HTML代码，比如
<pre lang="LANGUAGE">client.println("br /");</pre>
就是换一行。 用这两个函数就可以实现自己编辑文本。 比如我想加一句话“These are analog ports' readings”，换行，“Thanks！”，那么就在<em>break</em>前面加上
<pre lang="LANGUAGE">client.print("These are analog ports' readings");//这句输出第一句话
client.println("br /");//这句换行
client.print("Thanks!");//这句输出第二句</pre>
文字输出就这样解决！
<ul>
	<li>软件：步骤三，让Arduino发微博</li>
</ul>
<em><a title="脑震荡" href="http://www.naozhendang.com/" target="_blank">脑震荡</a></em>网站制作了一个微博App，让Arduino可以通过这个App发新浪微博。看看怎么玩！
它的App地址在这里：<a href="http://arduino2weibo.sinaapp.com/">http://arduino2weibo.sinaapp.com/</a>
首先，你需要在App上取得认证码。用你的新浪微博账号登录App，然后即可获取属于你的唯一认证码。保存好它，或者不要关闭窗口，过一会儿要把这段认证码放入Arduino的程序中。
接着，安装几个扩展库：<a href="http://arduino2weibo.sinaapp.com/code/Arduino2Weibo_0.2.zip" target="_blank">Arduino2Weibo扩展库</a>（由<em>脑震荡</em>开发）、<a href="http://gkaindl.com/software/arduino-ethernet" target="_blank">EthernetDNS扩展库</a>。安装方法是解压后放入Arduino IDE的Libraries文件夹中。
快完成了！把下面这段代码传入Arduino。别忘了更改IP地址和认证码。
<code>#if defined(ARDUINO) &amp;&amp; ARDUINO &gt; 18   // Arduino 0019 or later
#include
#endif
#include
//#include   Only needed in Arduino 0022 or earlier
#include 

// Ethernet Shield 设置
byte mac[] = { 0xDE, 0xAD, 0xBE, 0xEF, 0xFE, 0xED };

// substitute an address on your own network here
// 配置Arduino的局域网IP地址
// 请确保IP没有防火墙限制、没有和局域网内其他IP冲突
byte ip[] = { 192, 168, 2, 250 };

// 此处填写您的token和token secret (get it from http://arduino2weibo.sinaapp.com/)
Weibo weibo("YOUR-TOKEN-HERE","YOUR-TOKEN-SECRET-HERE");

// Message to post
// 由于Arduino IDE不支持敲中文，我做了个中文编码工具
// 想发送中文请访问 http://arduino2weibo.sinaapp.com/encode.php
//微博的内容（自己可以改）
char msg[] = "Hello Weibo!I'm Arduino! http://arduino2weibo.sinaapp.com";

void setup()
{
  delay(1000);//先等一会儿
  Ethernet.begin(mac, ip);//确定物理地址和IP地址
  Serial.begin(9600);//开始串口输出

  Serial.println("connecting ...");//正在连接
  if (weibo.post(msg)) {//如果连接成功
    int status = weibo.wait();//那就显示“稍等”
    if (status == 200) {//如果发送成功
      Serial.println("OK.");//那就显示“OK”
    } else {//否则……
      Serial.print("failed : code ");//显示“失败：错误代码是……”
      Serial.println(status);//“错误代码”
    }
  } else {//如果没连上
    Serial.println("connection failed.");//那就显示“连接失败”
  }
}
void loop()//为什么loop是空的呢？那是因为微博发一条就够了:D
{
}</code>
（以上一段代码出自<em>脑震荡</em>，为方便大家看懂我加了些注释）
给Arduino上电，这时打开微博，你的Arduino已经会说话了！
如果你想自定义微博内容，可以更改"<em>msg</em>"函数的值。<strong>需要注意的是</strong>，Arduino不支持中文，要发送中文，请使用<a href="http://arduino2weibo.sinaapp.com/encode.php" target="_blank">UTF-8 ENCODE工具</a>。
要想监控Arduino发微博的进度，可以打开Arduino IDE中的Serial Monitor。
<strong>最后提醒各位</strong>：Arduino IDE的版本千万要高于（或等于）0019，推荐Arduino 1.00。我一开始搞不定就是因为用的IDE版本过低！

关于Ethernet的入门内容就介绍到这里，更多有趣的内容，请查看网站里“MAKE&amp;Arduino”分类！