---
layout: post
title: 使用 Yubikey 放置 OpenPGP 密钥
date: '2016-05-14 11:39:41'
---

我使用OpenPGP进行简单的文件加密和git tag、commit签名有一段时间了，一直以来私钥都是和整个钥匙链一起放在本地Mac上。至于Yubikey，它刚支持U2F的时候我就听说了，但一直觉得应用面太窄而没有买。前几天偶尔看到他们网站上说推出了Yubikey 4，支持4096位OpenPGP密钥，感觉有点用，于是果断购入。

对重要文件加密的好处不用多说，我个人习惯把已经完成的项目资料tar之后加密，传到Onedrive之类的地方。git tag和commit的signing有助于保证代码完整性和防伪，而且这个功能前几天刚被GitHub支持。所以搞个自己的OpenPGP Key好处多多。但像这些RSA体系最脆弱的环节就是私钥的安全性。就像UNIX哲学“Whoever has physical access to the machine owns it."一样，拿到私钥和口令就可以直接以此身份签名、解密用对应公钥加密的文件、登陆服务器，想干嘛都行。因此，保障私钥的安全性是至关重要的。考虑到这方面，OpenPGP使用了MasterKey与SubKey组合使用的方式，用MasterKey签发SubKey，然后日常工作就用SubKey完成。SubKey出问题的时候，可以用MasterKey吊销出问题的SubKey。而因为MasterKey平时用不到，所以可以把它放在一台永不联网的机器上，从物理上保证它的安全。

OpenPGP对智能卡（Smart Card）的支持进一步加强了私钥的安全性。有别于把SubKey私钥存放在计算机本地，我们可以把私钥放在Smart Card上，然后随身带，这样只有从物理上获得了这个私钥，并在次数限制内输对PIN才能使用里面存放的私钥。而且，这样的方式事实上方便了日常使用，比如平时要在自己的笔记本和实验室的工作站用这些私钥，那就不用配置两台机器了，直接把Smart Card一插就好。

Yubikey存放OpenPGP Key就是通过把自己当作一张Smart Card。它里面有三个slot，可以分别放signing、encryption和authentication三个功能的Key。配置过程不麻烦，主要以下几步。这里假定已经有了Master Key，打算新建三个SubKey。
<h4>创建SubKey</h4>
这一步不麻烦，在gpg --edit-key里addkey就行。推荐三种用途各创建一个，以便利用上Yubikey里的三个slot。
<h4>备份SubKey</h4>
因为我们在把私钥放到YubiKey上后，本地机器上就不存在实际的私钥了，因此如果YubiKey丢了的话那这些私钥也就丢了，会很麻烦。用gpg --export-secret-key --armor备份一下，并存放在安全的离线位置。
<h4>导入SubKey</h4>
把YubiKey插进，进gpg --edit-key，用key &lt;index&gt;一次选中一个Key，然后keytocard就可以把它放到YubiKey上了。拷贝位置记得选和功能对应的那个slot。拷贝时要输入Admin PIN，初始值是12345678。

三个都拷贝完毕之后，quit并记得保存。这些操作完成后，私钥就不存在于本地机器上了，GPG会维护类似一个指针指向YubiKey，每次要用的时候就会尝试读Smart Card。
<h4>配置YubiKey</h4>
事实上，YubiKey作为一张智能卡，有不少信息可以配置。gpg --card-edit就可以进去。可设置的包括持卡人姓名、偏好语言、公钥地址、修改PIN（初始值是123456）等等。公钥地址建议配置一下，可以把公钥放到GitHub Gist上，然后把raw file链接配置进来。这样换到一台新机器上，只要fetch一下就可以自动配置公钥了，很方便。

YubiKey 4其实多了个额外的安全功能，就是在尝试访问存在卡上的密钥时，要求用户在它的电容传感器上按一下。这主要是为了让用户再次确认，正在访问卡上的机密信息，这样无论谁想使用卡上的私钥，都必须让用户得知，做到万无一失。
<h4>安装 PinEntry</h4>
在Mac上使用PGP Smartcard时，需要一个程序向用户请求PIN。Mac上Homebrew安装的GPG默认直接在terminal里请求，这会带来一些问题，比如一个程序在后台请求PIN时。而且，terminal里的PIN请求往往带着一些warning，看起来不是很舒服。所以，需要安装一个`pinentry-mac`。这样在请求PIN时，会出现一个GUI的弹出窗口。
