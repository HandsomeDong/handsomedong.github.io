---
title: Write-up-CTFHUB-web之SVN泄露、HG泄露
date: 2025-08-19 23:52:17
tags:
    - CTF
    - Web安全
    - Writeup
    - 信息泄露
    - SVN泄露
    - HG泄露
    - ctfhub
categories:
    - CTF
    - Web
    - Writeup
    - web
---
# 前言
SVN 是一款类似于 git 的版本控制工具，不过比 git 要简单易用，但是使用人数应该是比 git 少得多的，以前我在第一家公司时倒是用过，但是用得很少，主要是游戏研发部门的项目用到。

Mercurial（HG） 是一个免费、开源的分布式版本控制系统（Distributed Version Control System - DVCS）。它的名字意为“水银”，象征着轻快和灵活。其命令行工具的可执行文件名为 hg，是化学元素“汞”（Hydrargyrum）的缩写，也是对最初开发者 Matt Mackall 的致敬（hg 是他用户名的缩写），这个软件我也没有使用过。

# 工具
## dirsearch
项目地址：https://github.com/maurosoria/dirsearch

dirsearch 是一款基于 python3 的扫描工具，常用于暴力扫描页面结构，包括网页中的目录和文件

## dvcs-ripper
项目地址：https://github.com/kost/dvcs-ripper
dvcs-ripper 是一款基于 perl 的版本控制软件信息泄露利用工具，支持 SVN, GIT, Mercurial/hg, bzr 等

# SVN

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

# HG

打开页面是这样的

![0.png](https://s2.loli.net/2025/08/20/aVCxMf3qvEmk2PA.png)

老规矩，使用 dirsearch 扫描，查看请求成功的文件

![1.png](https://s2.loli.net/2025/08/20/oxeEXnQvtsY5Ik4.png)

使用 dvcs-ripper 工具下载 .hg 文件夹

```
perl /home/kali/tools/dvcs-ripper/rip-hg.pl -u http://challenge-1cb22b11b0567b80.sandbox.ctfhub.com:10800/.hg
```

全局搜索 flag 和 ctfhub

![3.png](https://s2.loli.net/2025/08/20/z3Bik4Tycv7whn6.png)

> .hg/store/fncache 存的文件索引数据，通过 fncache 可以快速定位到某个文件的位置，hg 会在路径前加 data/，并在结尾加 .i 表示这是一个索引，所以这里的 "data/flag_2180410243.txt.i"对应的路径就是 "flag_2180410243.txt"

直接请求该文件，拿到flag

```
curl http://challenge-1cb22b11b0567b80.sandbox.ctfhub.com:10800/flag_2180410243.txt
```

![4.png](https://s2.loli.net/2025/08/20/lSRr1C6oadwDNA3.png)



# 总结

信息泄露题可以先使用 dirsearch 工具确认哪些信息可能泄露了，再对症下药，使用相应的工具利用泄露的文件找到 flag 。