---
date: "2016-03-30T12:02:48Z"
aliases:
    - /2016/03/30/imported_from_old_blog_75.html
title: 配置FreeRadius和相关的证书信任链
---

寝室一直使用我的WNDR4300给全寝室提供Wifi接入。昨晚我突然想利用一下DD-WRT上自带的FreeRadius，然后用WPA2 Enterprise，然后给每个人一组用户名密码以便于管理。下面是配置过程和遇到并解决了的问题。

首先打开FreeRadius。DD-WRT给的配置界面比较傻瓜，看着配置就行。首先填写用于RADIUS服务器的证书，然后Generate Cert，路由器就会给自己签发一个证书。之后，Client里添加上要使用这个RADIUS服务器的主机的地址（其实也就是hocalhost），User里创建一下帐户就行。

之后，进到无线安全页面，认证模式选WPA2 Enterprise，加密算法务必选AES，鉴权服务器地址就填FreeRadius运行的主机的地址（其实也是localhost），Passphrase填刚才创建Client的时候写的那个。之后，就能正常运行了。

如果认为这样就能把WPA2 Enterprise无缝部署到所有设备上，那就大错特错了！事实上，昨晚搞定上述步骤之后，不同的设备上出现了不同的问题。而这些问题的核心都是自签发的证书不受信任。

首先描述一下不同设备上出现的问题。Mac OS X上的问题最小，只是在尝试登录时提示根证书不被信任，然后会提示把这个根证书加入系统信任列表。照做之后可以正常连上，但是为此，需要往系统根证书列表中添加一个随便自签发的证书，让人比较担心潜在的安全风险，至少如果我是用户，我是不可能愿意的。Windows上问题比较严重，因为Windows默认检查证书是否被信任，如果不信任，那就直接不连接。这个选项可以关掉，但是藏的非常深。iOS上连接时也需要手动把证书添加进信任列表，但是麻烦的是，这个过程似乎是不可逆的！要恢复原状，至少需要重置网络设置，相当麻烦，而且也有安全性隐患。只有Android上似乎没有任何提示就放行了（可是这才是最大的安全性问题吧！！）。

其实，利用IEEE 802.1X和EAP做鉴权时，操作系统根本无法通过证书完全确认主机的真实性。因为，在没有联网时，可以认为没有网络，没有DNS（故而无法确认该证书是否签发给该主机），甚至可以认为没有可靠的同步时钟（故而无法确认证书有效期）。所以理论上，使用一个自签发的证书用于RADIUS服务器并没有任何的问题，或者说，和使用一个权威CA签发的证书没有任何区别。事实上也是如此：无论用什么证书，在首次连接时都会提示用户人工确认证书正确性。然而，也有操作系统选择只信任权威CA签发的证书，否则连人工确认的机会都不给（如Windows）。而且，使用权威CA的证书好处在于，不用修改系统根证书列表，因而不会影响其他情况下的安全性。因此，配置一个具有完整信任链的证书用于RADIUS服务器是有意义的。

我使用的CA是StartSSL，他家提供免费Class 1 SSL证书。从CA那里获取证书的步骤很平常，但是把这些东西放到路由器上就比较麻烦。首先，我们要确认DD-WRT把各种配置文件放在哪里了：在我的WNDR4300上，是放在 /jffs/etc下，而不是Linux惯例的/etc下。/etc貌似是个只读的目录。然后需要稍微了解下FreeRadius的配置文件：grep一下发现全集中在eap.conf中。事实上，要给的只有Server Cert路径、Private Key路径，CA Cert路径和Private Key Passphrase。最后这个Private Key Passphrase我发现有点麻烦，因为每次重启都会被重置回DD-WRT的Web界面上（用于自签发证书）填的那个。我的解决办法是用在Web界面上填进新的Passphrase，然后重新Gen Cert，然后就不管Web界面了。

Server Cert需要一个完整的信任链，因为StartSSL的Intermediate CA似乎没有被Mac OS X识别，必须从StartSSL的Root CA开始走。所以Server Cert文件中，我们首先需要把三个证书：Root CA、Intermediate CA和Server Certificate合并到一起。这一步StartSSL帮我做了，retrieve下来的用于Nginx Server的就是bundle在一起的一个pem。然后，我把Private Key也放进同一个文件里了，之后Server Cert路径和Private Key路径填同一个就行。最后，把CA Cert路径指向下载下来的StartSSL Root Cert。重启路由器，验证可用即可。

配置的思路其实相当简单：尽量确保操作系统可以识别Server Cert并信任它（通过配一个完整的信任链）。

这样搞定后，在Mac OS X上首次连接时提示用户确认证书，但证书信息边上显示的不再是红色小叉，而是绿色小勾，表示操作系统信任此证书。Windows上首次连接时提示用户确认服务器指纹（无警告），之后一切正常。

唯一出问题的是iOS：连接时提示确认证书，但边上仍然显示一个红色“不被信任的”。我尝试给手机添加描述文件，把StartSSL Intermediate CA加进信任列表，也没用（其实是当然的，因为我在服务器上已经把信任链配好了）。Google多时后发现这似乎是iOS的一个安全机制。最后的解决办法是把Wi-Fi连接一并配好：在描述文件中添加一项Wi-Fi配置，然后选择WPA2 Enterprise并手动选择服务器证书。至此，所有设备都可以无缝使用WPA2 Enterprise，同时不会产生任何安全隐患。