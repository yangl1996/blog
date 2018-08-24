---
date: "2011-12-31T02:35:39Z"
title: Arduino+Ethernet Shield：远程控制
---

非常抱歉，图全部坏了

<blockquote>在任何角落控制你的家</blockquote>
<ul>
	<li>引言：用Ethernet Shield完成些更高级的事</li>
</ul>
Ethernet Shield让Arduino有了上网的能力。但这个能力绝对不止架个服务器、让Arduino发发微博那么简单。这次，试试远程控制Arduino吧！
<ul>
	<li>准备清单#1</li>
</ul>
为了让各位方便操作，我会在每一篇实验笔记中都加上准备清单

要尝试第一个实验，你需要准备
<ol>
	<li>Arduino Duemilanove 或 UNO</li>
	<li>Ethernet Shield</li>
	<li>一条USB线</li>
	<li>一条以太网线</li>
	<li>路由器</li>
	<li>Arduino IDE（集成开发环境）和需要用到的库</li>
	<li>一台Android系统的设备</li>
	<li>一枚LED</li>
	<li>一些线</li>
</ol>
<ul>
	<li>实验#1：控制LED</li>
</ul>
我建议各位先把软件写好。

我在网上找了一段代码，但是由于版本不同，无法正确运行。我只好手工修改了一下。下面放出修改版本（在Arduino 1.0上完全可用）
<code>// ARDUINO 1.0
// Edit by YangLei(yangl1996)
//
#include
#include
#define action_none -1
#define action_out_all 0
#define action_on_light 1
#define action_off_light 2
byte mac[] = { 0xDE, 0xAD, 0xBE, 0xEF, 0xFE, 0xBE }; //physical mac address
byte ip[] = { 192, 168, 1, 222 };
byte gateway[] = { 192, 168, 1, 1 };
byte subnet[] = { 255, 255, 255, 0 };
EthernetServer server = EthernetServer(80);
String readString = String(30); //string for fetching data from address
// arduino out
int pinOutPlight = 4;

// incoming GET command
String r_pinOnLight = "GET /?out=4&status=1";
String r_pinOffLight = "GET /?out=4&status=0";
String r_out_all = "GET /?out=all";
// current action
int current_action;
void setup(){
  //start Ethernet
  Ethernet.begin(mac, ip, gateway, subnet);
  delay(1000);
  pinMode(pinOutPlight, OUTPUT);
  digitalWrite(pinOutPlight, LOW);
  //enable serial datada print
  Serial.begin(9600);
  current_action = -1;
}
void loop(){
  current_action = -1;
  // Create a client connection
  EthernetClient client = server.available();
    if (client) {
 while (client.connected()) {
  if (client.available()) {
   char c = client.read();
   //read char by char HTTP request
   if (readString.length() <; 30)
   {
     //store characters to string
     readString = readString + c;
   }
   //output chars to serial port
   //Serial.print(c);
   //if HTTP request has ended
   if (c == 'n') {
    Serial.print(readString);
    // ****************************************************
     if(readString.startsWith(r_pinOnLight))
     {
     Serial.print("n ON UP n");
     current_action = action_on_light;
     }
     else if(readString.startsWith(r_pinOffLight))
     {
      Serial.print("n OFF UP n");
      current_action = action_off_light;
     }
     else if(readString.startsWith(r_out_all))
     {
  Serial.print("n ALLn");
  current_action = action_out_all;
     }
     else
     {
  Serial.print("n None n");
  current_action = action_none;
     }
    // ****************************************************
     // now output HTML data starting with standart header
     client.println("HTTP/1.1 200 OK");
     client.println("Content-Type: text/html");
     client.println();
    char buf[12];
    switch(current_action)
    {
    case action_out_all:
  client.print("{"ip" : "192.168.1.222", "devices" : [{ "type" : "light", "name" : "LED", "out" : "");
  client.print(pinOutPlight);
  client.print(""}");
  client.print("]}");
      break;
    case action_on_light:
      digitalWrite(pinOutPlight, HIGH);
      client.print("{"status" : "1" , "out" : "");
      client.print(pinOutPlight);
      client.print(""}");
      break;
    case action_off_light:
      digitalWrite(pinOutPlight, LOW);
      client.print("{"status" : "0" , "out" : "");
      client.print(pinOutPlight);
      client.print(""}");
      break;
    default:
      current_action = action_none;
    }

    // ****************************************************
     //clearing string for next read
     readString="";
     //stopping client
     client.stop();
   }
 }
    }
  }
}</code>
就是这些！

把软件写入Arduino！

接着连接硬件。从上面的程序中不难看出，你应该把LED连到Digital4口。很简单的工作。连好以后的样子应该是这样
<a href="http://yangl1996-wordpress.stor.sinaapp.com/uploads/2011/12/20111231-114157.jpg"><img class="alignnone size-full" src="http://yangl1996-wordpress.stor.sinaapp.com/uploads/2011/12/20111231-114157.jpg" alt="20111231-114157.jpg" /></a>
把网线插上，有关Arduino的工作就完成了！

下面开始连接Arduino。
打开DomoticHome，在Settings里设置好Arduino的IP地址，端口号一般不需改变。
点Sync，如果一切正常，会出现如下显示<a href="http://yangl1996-wordpress.stor.sinaapp.com/uploads/2011/12/20111231-114512.jpg"><img class="alignnone size-full" src="http://yangl1996-wordpress.stor.sinaapp.com/uploads/2011/12/20111231-114512.jpg" alt="20111231-114512.jpg" /></a>
如果看到了，那你离成功不远了！如果有问题，请仔细检查各项连接和设置，如果无法解决，可以评论此文章，我会回复的。其实我在连接Arduino的时候也遇到一些问题，总是连不上。有一个方法是使用USB给Arduino供电，我就是通过这个办法解决问题的。

要想控制你的LED，只要点Light1，进入后点解锁的图标即可！要关闭请点上锁的图标
<a href="http://yangl1996-wordpress.stor.sinaapp.com/uploads/2011/12/20111231-114859.jpg"><img class="alignnone size-full" src="http://yangl1996-wordpress.stor.sinaapp.com/uploads/2011/12/20111231-114859.jpg" alt="20111231-114859.jpg" /></a>
我的网站放在SAE，流入流出带宽需要节约一点，所以就暂时不放视频了。见谅！