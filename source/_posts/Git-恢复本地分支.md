---
title: Git恢复本地分支
date: 2019-04-27 00:52:45
tags:
    - Git
categories: 
    - Git
    - 工作中的Git
---
一般来说，我们开发新功能都会在develop分支下新建一个分支，命名为 **feature/新功能**，测试没问题后再合并到develop推到远程分支。由于我是新手，老大通常要我先别合并，先推到远程的新分支下等他review没问题后再合并到develop。

有强迫症的我，看到本地太多分支就不舒服，推到远程分支后直接就删掉本地分支，只留master和develop两个分支，干干净净舒舒服服！！！

![帅字贯穿了我的一生](https://i.loli.net/2019/04/27/5cc34c7e105f4.png)

万万没想到没想到老大也喜欢干干净净舒舒服服，但他不仅喜欢本地干干净净，也喜欢远程分支干干净净……然后今天早上问我这个功能的代码推到哪个分支了，我看了半天好像也找不到，老大说可能是他误删了……我……

![枯了](https://i.loli.net/2019/04/27/5cc34c7e15923.png)

原谅我是个菜鸡，Git可以恢复本地分支我都唔鸡，老大提醒我后我赶紧看了一下如何恢复本地分支，没想到如此简单！我只知道怎么在现有分支回滚代码，万万没想到连分支都能恢复！

![流下了没技术的泪水](https://i.loli.net/2019/04/28/5cc59b0db9ebe.png)

**仅需两步即可找回丢失分支！**
1.git log -g 找到删除分支前的commit_id
2.git branch recover commit_id  把代码恢复到新分支recover下

# 下面来演示全过程
## 1.新建并切换到新分支test
```
ls
git checkout -b test
```

![新建并切换到新分支test](https://i.loli.net/2019/04/27/5cc34a8b4584c.png)

## 2.新建文件随便写点东西
```
vim test.txt
```
我随便写了个 "test"

![新建文件随便写点东西](https://i.loli.net/2019/04/27/5cc34a8b487e8.png)

## 3.提交代码
```
git add .
git commit -m "Add test.txt"
git branch
```
可以看到现在有master和test两个分支

![提交代码](https://i.loli.net/2019/04/27/5cc34a8bb2edf.png)


## 4.切换回master分支并删除test分支
```
git checkout master
git branch -D test
git branch
```
可以看到现在只有master一个分支了

![删除分支](https://i.loli.net/2019/04/27/5cc34a8bcdea0.png)


## 5.git log -g查看提交记录
```
git log -g
```
可以从提交记录可以看到commit、commit的id及提交者等信息。通过conmit来找到你想要恢复的代码，所以一个好的commit很重要哦。

![查看提交记录](https://i.loli.net/2019/04/27/5cc34a8be96b9.png)

## 6.找到commit_id并恢复到新分支
```
git branch recover d0bd911dcbe57eb4ff1c99dd03b42da34a75640a

```
## 查看结果
```
git branch
git checkout recover
ls
cat test.txt
```
可以看到在recover分支和之前删除的test分支是一样的，文件一样，文件内容也一样，恢复分支成功！

![查看结果](https://i.loli.net/2019/04/27/5cc34a8beb6a2.png)

所以一定要好好commit啊，不然不管是回滚代码还是恢复分支，光是找节点就能让你找半天！