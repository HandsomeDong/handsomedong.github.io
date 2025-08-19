---
title: Write-up-CTFHUB-web之SVN泄露
date: 2025-08-19 23:52:17
tags:
    - CTF
    - Web安全
    - Writeup
    - 信息泄露
    - SVN泄露
categories:
    - CTF
    - Web
    - Writeup
    - web
---
# 前言
SVN 是一款类似于 git 的版本控制工具，不过比 git 要简单易用，但是使用人数应该是比 git 少得多的，这工具我也很少用，以前在第一家公司时倒是用过，但是我用得比较少，主要是游戏研发部门的项目用到。

# 工具
## dirsearch
项目地址：https://github.com/maurosoria/dirsearch

dirsearch 是一款基于 python3 的扫描工具，常用于暴力扫描页面结构，包括网页中的目录和文件

## dvcs-ripper
dvcs-ripper 是一款基于 perl 的版本控制软件信息泄露利用工具，支持 SVN, GIT, Mercurial/hg, bzr 等

# 解题

打开页面是这样的，提示我们 flag 在源代码里
![0.png](https://s2.loli.net/2025/08/20/FqeovAQhsWcg8Pl.png)

使用 dirsearch 工具扫描这个页面，得到扫描结果后，搜索 200 看看请求成功的文件

```
dirsearch -u http://challenge-776f8dc47b983ee8.sandbox.ctfhub.com:10800/
```

![2.png](https://s2.loli.net/2025/08/20/M27dWBkuiG1RNTa.png)

从上面的图中可以看到是 .svn 文件夹泄露了，因此使用 dvcs-ripper 工具还原文件夹

```
perl /home/kali/tools/dvcs-ripper/rip-svn.pl -u http://challenge-776f8dc47b983ee8.sandbox.ctfhub.com:10800/.svn
```

![3.png](https://s2.loli.net/2025/08/20/d1JPqvsYeXRNGhK.png)


用 vscode 打开工程，搜索一下 flag 或者 ctfhub，可以找到 flag

![4.png](https://s2.loli.net/2025/08/20/E8Gm7Ze4bBYRyHg.png)

# 总结

信息泄露题可以先使用 dirsearch 工具确认哪些信息可能泄露了，再对症下药，用相应的工具利用泄露的文件找到 flag 。