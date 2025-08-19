---
title: 'Write-up: CTFHUB web之Git泄露'
date: 2025-08-18 21:54:19
tags:
    - CTF
    - Web安全
    - Writeup
    - 信息泄露
    - Git泄露
categories:
    - CTF
    - Web
    - Writeup
    - web
---

# 前言
git应该是最主流的版本控制工具了，但是如果配置不当，可能会不小心把 .git 文件夹直接部署到生产环境。当然，这都2025了，不太可能会有开发出现这种错误了……

# 工具
## Githack
项目地址：https://github.com/lijiejie/GitHack
这是一个.git泄露利用脚本，可以通过.git文件夹下的文件，重建还原工程源代码。

下载后，用python2就可以运行该脚本
```
python GitHack.py http://www.openssl.org/.git/
```

# Log
ctfhub 的 git 泄露一共有3道，先来做第一道：Log

![image.png](https://s2.loli.net/2025/08/18/Pg46BCWhMVmY3al.png)


打开后就一个页面，显示“Where is flag?”
![image.png](https://s2.loli.net/2025/08/18/cktFWeN5zHpI1jQ.png)

直接用脚本下载并重建工程
```
python2 GitHack.py http://challenge-4b71d7d837e22605.sandbox.ctfhub.com:10800/.git
```
![image.png](https://s2.loli.net/2025/08/18/dKW8nlYpC6hax9S.png)


全局搜索 "ctfhub" 或者 "flag"，在源代码中搜不到，但是在git log相关文件中能看到提交记录有端倪，应该是添加了 flag 后又删掉了。

![image.png](https://s2.loli.net/2025/08/18/5QATtId8aDZfn6o.png)


用 git reset 回滚到 "add flag" 那一次提交后再全局搜索 "ctfhub" 或者 "flag"，可以在一个 txt 文件里找到 flag 了

```
git reset --hard fa0ee775a01e795f67ef4cb9d816ed52b0321473
```

![image.png](https://s2.loli.net/2025/08/18/eI5RoP8OFhJXl4d.png)

## Stash
前面的步骤和上面的一样，不过这次的txt文件里可就没有flag了
![image.png](https://s2.loli.net/2025/08/18/eHl9vw8rIpbZ7hy.png)

但是在 .git/logs/refs/stash 里面看到有个 flag 的日志，stash 是什么可以看这里 https://git-scm.com/docs/git-stash/zh_HANS-CN
![image.png](https://s2.loli.net/2025/08/18/s5igbSB9RnMH3xl.png)

用 git stash list 看看暂存的修改
```
git stash list
```
![image.png](https://s2.loli.net/2025/08/18/Q2EPmrHX3qIdA1S.png)


用 git stash pop 把暂存的修改还原后，可以看到txt文件里的flag了
```
git stash pop
```
![image.png](https://s2.loli.net/2025/08/18/m2K9CVPoBARfTlZ.png)

## Index
前几步步骤仍然和上面一样，但是txt文件直接就有flag了……
突然想到这几道题其实不应该直接用 Githack 工具还原工程，而是应该先用 dirsearch 扫描网站泄露的目录，扫出来发现有 .git 文件夹后，再用 Githack 工具还原工程，毕竟在真实的场景下，我们是不知道网站会有什么安全风险的，后续再更新一下用 dirsearch 该怎么做。

# 总结
碰到 git 信息泄露的情况，可以使用 Githack 工具还原工程，flag 可能存在于旧版本、别的分支、暂存区中，得细心找找！