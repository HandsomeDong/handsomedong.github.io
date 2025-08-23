---
title: XSS攻击练习
date: 2025-08-24 01:11:57
tags:
    - CTF
    - Web安全
    - Writeup
    - XSS
    - ctfhub
categories:
    - CTF
    - Web
    - Writeup
---
# 前言
今天学习 xss 攻击，在一个 xss 练习平台上玩了一下，感觉还是挺有收获的。
地址：http://test.ctf8.com/

![image.png](https://s2.loli.net/2025/08/24/9nJImVvXUBE8i45.png)

# 练习

## level 1

猜想这里直接回显 GET 参数里的 name

![image.png](https://s2.loli.net/2025/08/24/oc6lbDGvTWnRyuf.png)

构造 payload 为 ?name=&lt;script&gt;alert(1)&lt;/script&gt;，通过！

## level 2

这题看起来和上一题也差不多

![image.png](https://s2.loli.net/2025/08/24/FxDPb4yGmYrzuOl.png)



输入 &lt;script&gt;alert(&quot;七里翔&quot;)&lt;/script&gt; ，结果发现不行，看了看 HTML 源码，显示的部分明显是使用了 htmlspecialchars() 转换了特殊字符。

![image.png](https://s2.loli.net/2025/08/24/QBbsPdwt5Si4e2A.png)

虽然 h2 标签里展示部分的特殊字符被转换了，但是 input 标签里回显的内容却没有被转换，所以注入点便是 input 里面了。

这里先用 &quot;&gt; 闭合 input 标签，再注入 &lt;script&gt;alert(&quot;七里翔&quot;)&lt;/script&gt; 。输入 &quot;&gt; &lt;script&gt;alert(&quot;七里翔&quot;)&lt;/script&gt; ，通过！

## level 3

![image.png](https://s2.loli.net/2025/08/24/7WTuFfkcE2mbDAZ.png)

这题连 input 里的 value 也转换了，不过这里的 value 使用了单引号，并且输入单引号发现没有被转换。

![image.png](https://s2.loli.net/2025/08/24/iUD5XjNuhMfKl61.png)

使用单引号闭合 value ，再通过事件触发来注入，最后用 / 结束标签（/&#x27;&gt; 在属性型 XSS payload 中是合法写法）。输入 &#x27; onmouseover=alert(1) /，通过！

## level 4

![image.png](https://s2.loli.net/2025/08/24/81RCEnKuzhlIZi7.png)

输入 &quot;&gt; &lt;script&gt;alert(1)&lt;/script&gt; ，发现 &lt; &gt; 都被替换掉了。

![image.png](https://s2.loli.net/2025/08/24/tbzBuiZNnS76QeC.png)

那就用事件触发吧，输入 &quot; onmouseover=alert(1) / ，通过！

## level 5

![image.png](https://s2.loli.net/2025/08/24/JZAlkfyTO3iWrgV.png)

这下 on 被替换成 o_n 了，试了下 script 等标签，也被替换成了 scr_ipt。
![image.png](https://s2.loli.net/2025/08/24/WvsZtck4ExmFqC6.png)

尝试用用大小写绕过，输入 " OnMouseover=alert(1) / ， 发现全部都会被转成小写再替换。

尝试闭合这个标签，再另外构造一个 a 标签，输入 &quot;&gt; &lt;a href=&quot;javascript:alert(1)&quot;&gt;click me &lt;/a&gt; 。

最后的结果为 &lt;input name=keyword  value=&quot;&quot;&gt; &lt;a href=&quot;javascript:alert(1)&quot;&gt;click me &lt;/a&gt;&quot;&gt; ，点击 click me 后通过！

![image.png](https://s2.loli.net/2025/08/24/xLpXRINPsCHOa8G.png)

## level 6

![image.png](https://s2.loli.net/2025/08/24/ULI4jXD7PAYN9Gi.png)

用上一题的 payload，结果是 href 也被替换成了 hr_ef。

![image.png](https://s2.loli.net/2025/08/24/D9I2bsSMcUxwkie.png)

尝试用一下大小写混淆，结果为 &lt;input name=keyword  value=&quot;&quot;&gt; &lt;a Href=&quot;javascript: alert(1)&quot;/&gt;&quot;&gt; 。通过！

## level 7

![image.png](https://s2.loli.net/2025/08/24/ZDz6j3N4ewUxu5f.png)

尝试用上一题的 payload，即 &quot;&gt; &lt;a Href=&quot;javascript: alert(1)&quot;/&gt; ， 发现大小写混淆也无用，并且 href 和 script 都被直接替换掉了。

![image.png](https://s2.loli.net/2025/08/24/MyUci8vCaunVo3A.png)

尝试双写 href 和 script，输入 &quot;&gt; &lt;a hrHrefef=&quot;javascrscriptipt: alert(1)&quot;/&gt; 。通过！

## level 8

这一题稍微有点不一样了。

![image.png](https://s2.loli.net/2025/08/24/e1kJHf5F9jalQ4P.png)


输入 123321 看一下源码，可以看到这个友情链接 a 标签的 href 就是输入的内容。
![image.png](https://s2.loli.net/2025/08/24/8EoX1nkRq4iZzJP.png)

输入 javascript: alert(1) ，发现 javascript 被替换成了 javasc_ript 。

![image.png](https://s2.loli.net/2025/08/24/xob9p82UmVN6i1A.png)

尝试闭合 href ，再使用事件触发，输入 &quot; onclick=alert(1) / ，可以看到双引号被转换成 HTML 实体，onclick 也被替换成 o_nclick 。

![image.png](https://s2.loli.net/2025/08/24/EFerRLIK81mbUiC.png)

这意味着无法使用双引号闭合 href，也无法使用协议绕过（javascript:alert(1)）了吗？难道……难道真的止步于此了吗？

只好上网搜了搜别人的 write up，发现还能使用 HTML 字符实体来替换一些字符，HTML 字符实体转换：https://www.qqxiuzi.cn/bianma/zifushiti.php。
这里随便把 javascript 里的 c 字符替换成 &#x26;&#x23;&#x78;&#x36;&#x33;&#x3B; ，输入 javas&amp;#x63;ript:alert(1) 。通过！（感觉这个操作有点骚啊！）

![image.png](https://s2.loli.net/2025/08/24/urvGneBg8mFw94Y.png)

## level 9

![image.png](https://s2.loli.net/2025/08/24/VpqrLtg7juBDh1S.png)

这一关会先校验输入是否是一个合理的链接地址，如果不是，则直接把 href 里的内容替换成"您的链接不合法？有没有！"

![image.png](https://s2.loli.net/2025/08/24/kH1zJISifqpBRZs.png)

尝试在上一题的 payload 基础上，加上注释一个链接，输入 javas&#x63;ript:alert(1) //http://www.baidu.com ，通过！看来这个校验，只校验是否包含链接，而不是校验整个输入是否是一个合理的链接！

![image.png](https://s2.loli.net/2025/08/24/5ish93YQovaJKTA.png)

## level 10

这关没有输入框，查看源码发现有隐藏的表单。

![image.png](https://s2.loli.net/2025/08/24/mIs6wKgQrYZqeXD.png)

尝试手动构造表单内容，请求 http://test.ctf8.com/level10.php?keyword=well%20done!?t_link=1&t_history=2&t_sort=3 发现 t_sort 内容会回显到输入框。

![image.png](https://s2.loli.net/2025/08/24/6y5UBb89lj71rPN.png)

构造 payload 到 t_sort 里，输入 http://test.ctf8.com/level10.php?keyword=well%20done!?t_link=1&t_history=2&t_sort=%22%20%20onmouseover=%22alert(1)%22%20type=%22text 。通过！

![image.png](https://s2.loli.net/2025/08/24/oeJLwR3BuXhYNUW.png)

## level 11

一进入这一关，就看到隐藏的表单输入框里回显了一个链接，这个不正是我上一关的链接吗？所以猜测这里是回显了 http header Referer 的信息，所以这里可能得用到 BurpSuite 了！

![image.png](https://s2.loli.net/2025/08/24/LUnrNaOqxMuDvdo.png)

刷新一下页面，构造一下 header 里的 Referer 为 " onclick=alert(1) type="text 。通过！

![image.png](https://s2.loli.net/2025/08/24/bPHLXuNFnUMZavq.png)

![2.png](https://s2.loli.net/2025/08/24/1hsRIlaBmFbpnVg.png)

## level 12 

这一关和上一关大同小异，只是把 Referer 换成了 User-Agent ，一样的套路，不过多赘述。

![image.png](https://s2.loli.net/2025/08/24/KcNtziXhMvR7JQ1.png)

## level 13

一样，只不过这次是 Cookie ，不过多赘述。

![image.png](https://s2.loli.net/2025/08/24/82MXupxhZwYeiH3.png)


## level 14

这题是真不会了，等后续有时间再上网找一些大神的 write up 观摩一下。

![image.png](https://s2.loli.net/2025/08/24/iAM1LEQuKc3Dn9Z.png)

## level 15

![image.png](https://s2.loli.net/2025/08/24/XvMyznHGVRaep8k.png)

这一关直接看源码，可以看到里面这个 span 的 class 有用到 src 参数，查了查 ng-include ，这是 AngularJS 的指令，有点类似于文件包含。https://www.runoob.com/angularjs/ng-ng-include.html

![image.png](https://s2.loli.net/2025/08/24/NDFsG5KZYfHPwOr.png)

由于这个包含的文件需要与当前域名一致，所以可以考虑考虑用前面关卡的文件，再把 payload 注入到前面的文件链接中。所以我这里构造的 payload 是 &#x27;level1.php?name=&lt;img src=x onerror=alert(1)&gt;&#x27; 。通过！

![image.png](https://s2.loli.net/2025/08/24/tahcjSzP9Jb5YGk.png)

## level 16

![image.png](https://s2.loli.net/2025/08/24/ZFrkQVTiIs2B5ow.png)

![image.png](https://s2.loli.net/2025/08/24/zL5ytjE8267HVsa.png)

尝试直接注入，?keyword=&lt;script&gt;alert(1)&lt;/script&gt; ，script 和 / 都被替换成了 &amp;nbsp; ，大小写混淆也失败。

![image.png](https://s2.loli.net/2025/08/24/rt4QgPW62Ue75K8.png)

那就尝试用 img + error 事件触发，?keyword=&lt;img src=x onerror=&quot;alert(1)&quot;&gt; ，发现空格也被替换了。

![image.png](https://s2.loli.net/2025/08/24/tbjnfxSL1IF9Qwv.png)

尝试用 %0d 代替空格来绕过检查，?keyword=&lt;img%0dsrc=x%0donerror=&quot;alert(1)&quot;&gt; 。通过！

## level ……

后面几关好像都没法显示，貌似是 flash 的，后续有时间再做吧，现在暂时不了解 flash xss。

![image.png](https://s2.loli.net/2025/08/24/dpOB1J8QSqbKT2W.png)

# 总结
1. 先确认注入点，再开始尝试注入
2. 常见的注入方式有：&lt;script&gt; 直接执行、利用 HTML 标签自带的事件（如 img + onerror、 iframe + onload、 input + onclick 或 onmouseover 等）、meta 标签刷新跳转等（不过这个练习中没有这种题）
3. XSS 绕过常用方法有：大小写绕过、编码绕过、闭合标签绕过、注释绕过、其它符号绕过、双字符绕过、事件绕过等等