---
layout: post
title: 分析北京大学统一身份验证系统的工作机制
date: '2015-03-10 10:06:33'
---

最近写一个自动将学期选课结果转换成 .ics 日历交换文件的 Python 脚本。一开始的打算是用户自行下载包含选课结果的 HTML 文件，后来为了提高易用性，希望做成一个 Web App，让用户输入用户名和密码，由服务器代为下载。所以需要服务器代替用户登录选课系统。北京大学选课系统和北大的其他诸多网络服务一样，使用叫做“北京大学统一身份验证”的服务。所以，我需要对这个流程进行反向工程，并用 Python 代码模拟这一过程。

本文主要分析北京大学统一身份认证（IAAA）的工作机制，不涉及具体的 Python 实现。如果有需要，可以看看这个 Web App 的代码：<a href="https://github.com/yangl1996/schedule-parser" target="_blank">https://github.com/yangl1996/schedule-parser</a>。

为了方便地发送 HTTP 报文，我采用了 Requests 模块。核心库中的 urllib2 使用起来有些反人类，而 Requests 用起来相当顺手。其实一开始我的打算是，让脚本模拟浏览器的行为，也就是填写表单、执行 JavaScript 等一系列工作，这样可以不用研究具体的 HTTP 报文传输。但是，适用于 Python3 且支持 JavaScript 执行的相关库并没有，所以我开始尝试对这个身份验证流程进行反向工程。

Google 一番，并没有找到北大统一身份认证的相关技术资料，只好先从网页源文件开始着手。

当访问 elective.pku.edu.cn 时，用户会被跳转到<code>https://iaaa.pku.edu.cn/iaaa/oauth.jsp?appID=syllabus&amp;appName=学生选课系统&amp;redirectUrl=http://elective.pku.edu.cn:80/elective2008/agent4Iaaa.jsp/../ssoLogin.do</code>这个页面。这个过程并不是通过 HTTP 3xx 响应完成，而是 JavaScript 完成的操作，所以 Requests 没法处理，需要直接访问上面这个新 URL。

查看这个页面，很容易就可以发现，当用户单击“登陆”时，执行的是<code>oauthLogon()</code>这个函数。在 JavaScript 源码中寻找，可以发现函数的内容：

```
function oauthLogon () {
if($("#user_name").val()=="" || $("#user_name").val()=="学号/职工号/北大邮箱") {
//$("#msg").text("账号不能为空");
$("#msg").html("账号不能为空");
$("#user_name").focus();
}else if($("#password").val()=="" || $("#password").val()=="密码") {
//$("#msg").text("密码不能为空");
$("#msg").html("密码不能为空");
$("#password").focus();
}else if($("#code_area")[0].style.display=="block" &amp;&amp;
($("#valid_code").val()=="" || $("#valid_code").val()=="验证码")) {
//$("#msg").text("验证码不能为空");
$("#msg").html("验证码不能为空");
$("#valid_code").focus();
}
else { //document.myForm.submit();
if($("#remember_check")[0].checked==true){
setCookie("userName",$("#user_name").val());
setCookie("remember","true");
}
else{
delCookie("userName");
delCookie("remember");
}
$("#msg").text("正在登录...");
$.ajax('/iaaa/oauthlogin.do',
{
type:"POST",
data:{appid: $("#appid").val(),
userName: $("#user_name").val(),
password: $("#password").val(),
randCode: $("#valid_code").val(),
redirUrl:redirectURL
},
dataType:"json",
success : function(data,status,xhr) {
var json = data;
if(true == json.success){
if(redirectURL.indexOf("?")&gt;0)
window.location.href = redirectURL+"&amp;rand="+Math.random()+"&amp;token="+json.token;
else
window.location.href = redirectURL+"?rand="+Math.random()+"&amp;token="+json.token;
}
else{
$("#msg").html(""+json.errors.msg);
$("#code_img")[0].src="/iaaa/servlet/DrawServlet?Rand="+Math.random();
if("账号未激活"==json.errors.msg){
window.location.href = "https://iaaa.pku.edu.cn/iaaa/activateAccount.jsp?Rand="+Math.random()+"&amp;activeCode="+json.activeCode;
}
else if("用户名错误"==json.errors.msg){
$("#user_name").select();
if(true==json.showCode){
$("#code_area")[0].style.visibility="visible";
$("#remember_area")[0].style.top="-5px";
$("#submit_area")[0].style.top="0px";
}
}
else if("密码错误"==json.errors.msg){
$("#password").select();
}
else if("验证码错误"==json.errors.msg){
$("#code_area")[0].style.visibility="visible";
$("#remember_area")[0].style.top="-5px";
$("#submit_area")[0].style.top="0px";
$("#valid_code").select();
}
}
},
error : function(xhr,status,error) {
$("#msg").html("查询时出现异常");
$("#code_img").attr("src","/iaaa/servlet/DrawServlet?Rand="+Math.random());
}
});
}
}
```

最开始一段，执行的是确认用户输入了密码和学号，如果需要验证码的话，填写了验证码。真正执行登陆信息传输的是这一段：

![](/content/images/2016/05/code.png)

看上去是一个 OAuth 认证，用了 AJAX 框架，不过这两个点我都不了解具体细节。不过要看懂 JavaScript 代码很容易。这应该是一个 HTTP POST 报文，传输的内容有 <code>appid, userName, password, randCode, redirUrl</code>。根据上下文稍加理解，可以发现 randCode 就是验证码，而其他字段基本上就是本意。下面要做的就是确定 appid、redirURL 的内容。

看一看这个页面的 URL，就会发现 URL 里自带了 appid 的值：<code>syllabus</code>。redirURL 也在 URL 里有：<code>http://elective.pku.edu.cn:80/elective2008/agent4Iaaa.jsp/../ssoLogin.do</code>。所以这些字段的值都可以确定了。至于 randCode，我猜测随便发一个整数就行。试验得出我的猜测是正确的。

HTTP POST 报文的内容确定好了，下面要处理的是服务器返回的请求。JavaScript 源码的下文提到了 JSON。这个我还有点熟，所以很快写好了解析的代码。服务器在收到客户端发送的 HTTP POST 报文之后，会进行验证，并把验证结果以 JSON 格式发回客户端。分析下文，可以发现，如果验证成功，那会返回状态值“success”和一个 Token。如果不成功，会返回对应的状态值。代码下文还给出了验证成功之后，要跳转到哪一个 URL。URL 中包含了 Token 和一个随机数。随机数我猜测是用来唯一标记会话用的。

用 Python 实现上述流程，工作正常。大概只需要10-20行代码。

总结一下北京大学统一身份认证（IAAA）的工作原理。

首先，应用将用户引导到 IAAA 界面。用户填写好用户名、密码，JavaScript 将用户名、密码，连同要登录的应用的 Unique ID、登录之后要跳转的 URL、验证码，一起发给 IAAA 服务器。服务器做验证，如果验证成功，会返回一个 Token。用户以这个 Token 作为验证成功的凭证，成功登陆应用。这整个流程确实基本是 OAuth 的流程。

那么事实上，北京大学 IAAA 是个半开放的系统。也许可以利用这个半开放的框架，实现类似“使用北京大学账号登录应用”的功能。