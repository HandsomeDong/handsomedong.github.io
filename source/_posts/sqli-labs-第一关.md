---
title: sqli-labs 第一关
date: 2025-08-09 16:11:57
tags:
    - CTF
    - Web安全
    - Writeup
    - sqli-labs
    - sql注入
categories:
    - CTF
    - Web
    - Writeup
    - sqli-labs
---
# 前言

> 最近也是玩上了CTF啊，不过CTF涉及到的知识点可真多，先从最简单的web安全-SQL注入学起吧！听说 SQLi Labs 是一个开源且全面的 SQL 注入靶场，那就拿这个来练练 SQL 手工注入吧！同时也学学 sqlmap 的使用。

# 手工注入
## 判断是否存在 SQL 注入
1. 一进来就提醒输入一个数字的id值

![image.png](https://s2.loli.net/2025/08/09/zmUvZL6F91Gatuh.png)

2. 输入发现会回显内容，输入不同的id值，会回显不同的内容，所以我们后端应该就是根据我们输入的值来查询数据了。（这里推荐使用 chrome 的 HackBar 插件，在浏览器直接输入的话，特殊符号老是被编码，不太好看）

![image.png](https://s2.loli.net/2025/08/09/G1D9AYaqi7KMbEC.png)

3. 接下来判断是否存在注入，是字符型注入还是数字型注入
直接输入 1 是没问题的，但是输入 1' 就报错了，所以这应该是字符型注入。

![image.png](https://s2.loli.net/2025/08/09/hVKdEtHfIvbL9C5.png)

![image.png](https://s2.loli.net/2025/08/09/Nj87fe3Z5uaOt2l.png)

4. 进一步验证猜想
为了进一步验证这是字符型，可以使用 AND 1=1、 AND 1=2 查看结果。


![image.png](https://s2.loli.net/2025/08/09/426VUfPjYKmu5DW.png)

![image.png](https://s2.loli.net/2025/08/09/YR9xturkiwEf5FA.png)
从上面2张图可以确定这不是数字型注入了，因为 AND 后面的条件没生效。

![image.png](https://s2.loli.net/2025/08/09/oPbjvZQrVqL924u.png)

![image.png](https://s2.loli.net/2025/08/09/2CaEwxKeHSVtd3s.png)
从上面2张图可以确定这是字符型注入，使用单引号闭合了前面的查询条件，因此 AND 后面的查询条件生效了，所以2次查询结果不一样。

## 联合注入爆数据
1. 确定 SELECT 列数
接下来就是使用联合注入爆各种数据了，不过在此之前需要先确定一下 SELECT 的内容有多少列，因为 UNION 的列数需要一致。
使用 ORDER BY 来确认列数，ORDER BY 4 时报错了，所以 SELECT 的列数是3。

![image.png](https://s2.loli.net/2025/08/09/lMm5iyFq1bCQzJH.png)

2. 爆显示位
输入一个不存在的id，把后面的内容 UNION 上去，确认显示位。
可以看到回显的内容是第二、第三列。

![image.png](https://s2.loli.net/2025/08/09/aqgoCQbSxNH2RGj.png)

3. 爆库名
接下来就可以爆整个数据库实例的库名了，先使用 database() 查看当前查询的库，以及查询 information_schema.schemata 的 schema_name 列获取所有库名。

![image.png](https://s2.loli.net/2025/08/09/76edbohSy5rZKU2.png)

4. 爆表名
查询 information_schema.tables 获取 security 库下的所有表名。

![image.png](https://s2.loli.net/2025/08/09/6lsrvCjtez3Rxnc.png)

5. 爆字段
当前显示了用户名、密码，这些数据肯定存在 users 表中，所以查查这张表有什么字段吧！
可以看到表中的字段有 id、username、password

![image.png](https://s2.loli.net/2025/08/09/zFs6qEQfJaTjCYx.png)

6. 爆数据
直接查询 users 表的 username 和 password

![image.png](https://s2.loli.net/2025/08/09/m6VIKUxEyejuX8t.png)

# sqlmap
## 使用 sqlmap 注入
sqlmap github: https://github.com/sqlmapproject/sqlmap

1. 先查询所有库

```
sqlmap -u http://192.168.0.120:10000/Less-7/?id=1 --dbs
```
![image.png](https://s2.loli.net/2025/08/09/1T8fLmZjkMVv2W3.png)

2. 查询 security 下的表

```
sqlmap -u http://192.168.0.120:10000/Less-7/?id=1 -D security
```

![image.png](https://s2.loli.net/2025/08/09/MsNDXunrwGg6tlo.png)

3. 查询 users 表数据

```
sqlmap -u http://192.168.0.120:10000/Less-7/?id=1 -D security -T users --columns --dump
```

![image.png](https://s2.loli.net/2025/08/09/5KaN9YlTciSrXWL.png)