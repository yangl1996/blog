---
layout: post
title: 当完成一个reCAPTCHA验证码
date: '2015-01-03 15:58:48'
---

CAPTCHA就是验证码。它并不是一个单词，而是一长串单词的缩写：Completely Automated Public Turing test to tell Computers and Humans Apart（全自动区分计算机和人类的图灵测试）。我们总是抱怨验证码又麻烦又费时，而且验证码越来越难正确完成了。不过换个方向想，所有人花在验证码上的时间和脑力劳动加在一起也是很可观的。如果把这些脑力劳动变得有意义，那也是很大的一笔资源。

reCAPTCHA服务的思路最早由CMU提出。通过在验证码中放置OCR无法识别的古籍扫描件、街景图片中的文字、需要human-labeling的机器学习数据，人们所做的验证码就有了意义。这个做法有点众包的感觉。在填写一个reCAPTCHA验证码时，可能就顺便把一本绝版古书中的一个单词数字化了，也可能恰好将自家门前的路牌变成了文字，还可能帮助构建了一个机器学习数据集。填个验证码也算是为人类作出了一点小小贡献。

![](/content/images/2016/05/Recaptcha_anchor-2x.gif)
No CAPTCHA reCAPTCHA

现在reCAPTCHA服务由Google提供，于是带上了一些更不错的功能和体验。比如Google会根据自己的数据，来判断当前来源IP是一个bot的可能性，从而自动显示适当难度的验证码。最近还推出更加逆天的“No CAPTCHA reCAPTCHA”，大部分用户只需要在复选框上打一个勾就能够直接通过测试，而系统通过该用户的来源IP和鼠标在复选框上的移动来判断这是bot的可能性。

![](/content/images/2016/05/reCAPTCHA_OldAPI.png)
左侧是待识别图片，右边是Control Word

通过同时验证两个单词，reCAPTCHA可以保证识别的准确性。一个单词是已知答案的control word，真正用于验证用户的身份，另一个则是待识别的图片，用户提交之后就相当于做了一次human-labeling。对于比较难的图片，系统会进行多次测试以保证准确度。这些被用户识别的图片做扭曲、反色等等处理，又会循环作为control word使用。通过这个系统做文字识别，达到了99.1%的准确度。

所以下次填完reCAPTCHA，不妨愉快地享受一下小小的成就感w