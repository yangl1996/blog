---
date: "2015-04-28T11:49:23Z"
aliases:
    - /2015/04/28/imported_from_old_blog_62.html
title: 申请 Google Summer of Code 2015
---

关于 GSoC，本来去年就很想和 <a href="http://jueast.com" target="_blank">Jueast</a> 和 <a href="http://www.mewinnie.com" target="_blank">Winnie Zhou</a> 一起组队参加。但是去年知道的时候已经快八月份了。GSoC 的申请实际上是3月份左右就开始的。今年三月，Winnie Zhou 告诉我们 GSoC 2015 开始接收 Student Applications 了。可是一看规则才发现，这个活动是只允许以单人为单位报名 / 参加的，只好单独申请。

GSoC 这个活动，实际上是在校大学生为开源组织提供代码（是否 merge 进入 master branch 另说），然后 Google 作为组织方，为学生和开源组织支付一笔“工资”。今年的奖励是对每个项目，学生 $5500，开源组织 $500。学生需要在暑假（其实还有5月底和6月份）完成一个具有一定规模的项目，用开源协议授权，然后提供给开源组织。每个学生会被分配一个很厉害的导师，解决编码过程中的各种问题。我感觉参加这个活动除了能练习较大型项目的编写、拿到一笔不错的奖励之外，应该还是一个融入开源项目的好机会。之前关于开源项目，我只给 <a href="https://github.com/tryghost/Ghost" target="_blank">Ghost</a> 和 <a href="https://github.com/digibart/upscuits" target="_blank">Upscuit</a> 做过一些中文翻译和提一些 bug，想要是能给开源社区贡献一些代码也是极好的。

申请时，每个学生可以提交复数的 Proposals，所以我和 Jueast 一开始都打算广撒网。参加的开源组织有几百个，一个个看完全看不过来，所以我找了自己用过的几个和感觉比较有意思的几个，包括 Python、CentOS、OpenCV，挑了4个项目准备写 Proposal。但其实写了一个就发现写起来还是很费劲的，所以把其他3个项目都放弃，专心写 CentOS 的这个。

CentOS 是 Linux 的一个发行版，从 RedHat 改来的。我的几台服务器目前运行的都是它。从 <a href="http://wiki.centos.org/GSoC/2015/Ideas" target="_blank">CentOS 今年的 Idea List</a> 上找的是比较有意思，又比较有自信的 <a href="http://wiki.centos.org/GSoC/2015/Ideas#docs-toolchain">Implement and create new documentation toolchain</a>。目前 CentOS 用 Wiki 来存放文档，但这个 Wiki 编辑比较不灵活，需要编辑者先去邮件列表里申请和描述。但是邮件列表这种东西现在除了开源社区和一些专业社区之外实在很少人用（还有 IRC，都是开源社区的爱好），所以对一般的贡献者来说门槛挺大。这个模式效率也比较低。因此出现了这个 Idea，希望底层用 Git，然后 GitHub 作为一个比较友好的介面，在这个基础上构建新的文档系统。

我写的 Proposal 大约2600词，分为以下几部分：Main Idea、Expected Results、Details、Main Advantages、Realization 和 Summary。第一版本实际上只包含了 Main Idea，提交之后马上就有 mentor 来 comment 要求加 detail 和 expected results。加上之后过了几天，收到新的 comment 建议添加计划的实现方法和用到的包。申请关闭之后，mentor 发来邮件，认为我的 proposal 规模略大，不适合一个“暑期项目”，所以建议我用现成的开源项目实现部分功能。我再作修改，并在 proposal 中多次指出“项目规模适合一个暑假完成”之后定稿。

修改 proposal 过程中我在邮件列表里发起过一两次讨论，也有同样申请这个项目的学生和 mentor 参加讨论。讨论中有一个印度人。之前两个月，我在邮件上指导一个印度人做 APRS Weather Box，感觉阿三的英语一般啊，结果这次这个印度人的英语很棒，表达什么的简直和美国人靠拢。相比之下我的邮件用词就略显简单啦&gt; &lt;。参加邮件列表的讨论是开源组织评价各个学生的重要一部分，因为这体现学生是否适应开源组织的开发环境。

最后大概只剩下我和阿三来抢这个项目。我们也都意识到要完美地完成这个项目，一个人似乎不太够，因为光设计底层架构就够呛。幸运的是，前几天 mentor 发来邮件，说 CentOS 的 student 名额有多，可以让我们俩分别进行一部分内容，"work parallel"。我二话没说就回"Good idea!"，过了一天阿三也回"OK"，还建议我们几个在 hangouts 上好好讨论一下。

今天一起床就看 GSoC 的页面，发现自己的 proposal 被接受啦！这个夏天应该要写不少代码了。

总结一下申请的经验：
<ol>
	<li>少而精地写 proposal，每一份都写得很详细实在没精力</li>
	<li>尽量早与自己喜欢的开源组织接触，甚至可以鼓动他们作为 organization 参加 GSoC</li>
	<li>Proposal 里尽量多写细节、实现方式、优点</li>
	<li>快速回应 mentor 的 comments</li>
	<li>参加开源组织的邮件列表讨论，要是 IRC 也参加就更好</li>
	<li>挑自己喜欢的 idea 类型</li>
	<li>个人编程历史不是很重要，我只在 proposal 里列出了现在所在学校、院系和个人主页地址</li>
</ol>