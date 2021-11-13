---
title: 记一次简单的服务启动GC优化
date: 2021-01-31 23:31:36
tags:
    - JVM优化
categories: 
    - Java
    - JVM
---
# 前言
今天上线一个项目的时候在日志里发现项目启动的时候频繁GC，花了点时间分析了一下并且调整了一下JVM参数。

# 症状
项目启动

```c
java -server -Xmx1024m -XX:MaxDirectMemorySize=512M -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -Xloggc:gc.txt -jar xxxxxxxx.jar
```

gc.txt
![GC记录](https://img-blog.csdnimg.cn/20201203194421400.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEwMTE1NQ==,size_16,color_FFFFFF,t_70)
好家伙，一启动就来9次 Young GC ，2次 Full GC。

现在先用jinfo看看配置参数
![查看JVM参数](https://img-blog.csdnimg.cn/20201203195944270.png)
注意几个重要参数：
```C
-XX:InitialHeapSize=62914560		//堆初始值
-XX:MaxHeapSize=1073741824	//堆最大值
-MaxDirectMemorySize=536870912	//虚拟机外最大内存
```



# 优化
## Full GC
先来分析这两次Full GC以及它们前面的Young GC

```c
2.536: [GC (Metadata GC Threshold) [PSYoungGen: 19883K->3584K(128512K)] 27143K->12014K(169472K), 0.0158155 secs] [Times: user=0.03 sys=0.00, real=0.01 secs] 
2.552: [Full GC (Metadata GC Threshold) [PSYoungGen: 3584K->0K(128512K)] [ParOldGen: 8430K->8782K(32256K)] 12014K->8782K(160768K), [Metaspace: 20409K->20409K(1069056K)], 0.0844666 secs] [Times: user=0.16 sys=0.00, real=0.09 secs]
4.965: [GC (Metadata GC Threshold) [PSYoungGen: 109848K->8697K(247296K)] 119422K->19608K(279552K), 0.0278398 secs] [Times: user=0.03 sys=0.01, real=0.03 secs]
4.993: [Full GC (Metadata GC Threshold) [PSYoungGen: 8697K->0K(247296K)] [ParOldGen: 10910K->14513K(47104K)] 19608K->14513K(294400K), [Metaspace: 33512K->33512K(1081344K)], 0.1136122 secs] [Times: user=0.20 sys=0.00, real=0.11 secs]
```
可以看到两次Full GC都是由于Metadata GC Threshold造成的。我这里用的是JDK8，参数里没有明确指定metaspace的初始值和上限，这个时候初始值应该是默认的 21M 。问题不大，那我就把它的初始值和上限调大一点吧，调到64M。

措施：
**追加元空间初始值**

```c
java -server -XX:MetaspaceSize=64m -XX:MaxMetaspaceSize=64m -Xmx1024m -XX:MaxDirectMemorySize=512M  -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -Xloggc:gc.txt -jar xxxxxxxx.jar
```

这个时候再查看gc.txt
![gc.txt](https://img-blog.csdnimg.cn/2020120320223724.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEwMTE1NQ==,size_16,color_FFFFFF,t_70)
可以看到已经没有Full GC了，不过还是有8次Young GC。

## Young GC
其实从我的启动参数里面就可以看到，**堆的容量我只指定了最大值(-Xmx1024m)**，并没有指定初始值。所以可以在配置参数里面看到 -XX:InitialHeapSize=62082048，即**堆的初始容量只有60m左右。**

而JVM默认的新生代和老年代空间占比为 **1 : 3**，所以新生代在一开始的时候只有15m的内存空间，因此JVM一开始在15m的时候就发生了Young GC。

所以现在要追加新生代和堆的初始值，新生代、老年代的空间占比我就不调了，使用默认的 1 : 3 即可，而堆的初始值调成1024m，这样新生代的初始值就是256m了。

措施：
**追加堆的初始值**

```c
java -server -Xms1024m -XX:MetaspaceSize=64m -XX:MaxMetaspaceSize=64m -Xmx1024m -XX:MaxDirectMemorySize=512M  -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -Xloggc:gc.txt -jar xxxxxxxx.jar
```

现在再来看看gc.txt
![gc.txt](https://img-blog.csdnimg.cn/20201203203514210.png)
可以看到只有两次Young GC了，这两次Young GC是在新生代对象准备超过256m的时候发生的……

**好家伙，既然如此，那我也不讲武德了，我现在就把堆初始值和最大值都调成100G。**
![过分](https://img-blog.csdnimg.cn/20201203204915316.png#pic_center)
好吧，公司穷，只能买得起4G的服务器。
![哭了](https://img-blog.csdnimg.cn/20201203205647704.png#pic_center)


# 总结
这只是一次很简单的JVM启动参数优化，平时启动应用的时候还是得多注意注意堆的初始值和最大值配置，元空间这部分也不能忽略了。