---
title: JDK7HashMap多线程扩容导致的死循环问题
date: 2020-11-14 21:46:19
categories: 
    - Java
    - 基础
---
# 前言
<font color=#999AAA >JDK8以前的HashMap，多线程扩容的时候可能会出现死循环，这个问题在JDK8得到了修复。本翔看了大半天JDK7HashMap扩容源码找这个问题，所以写篇博客记录记录。

# 源码
先来看看JDK7的HashMap扩容相关源码。
```
	//扩容
	void resize(int newCapacity) {
        Entry[] oldTable = table;
        int oldCapacity = oldTable.length;
        if (oldCapacity == MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return;
        }

        Entry[] newTable = new Entry[newCapacity];
        transfer(newTable, initHashSeedAsNeeded(newCapacity));
        table = newTable;
        threshold = (int)Math.min(newCapacity * loadFactor, MAXIMUM_CAPACITY + 1);
    }
    
    void transfer(Entry[] newTable, boolean rehash) {
        int newCapacity = newTable.length;
        //遍历旧桶数组
        for (Entry<K,V> e : table) {
            while(null != e) {
            	//先记录下一次循环要插入的节点
                Entry<K,V> next = e.next;
                if (rehash) {
                    e.hash = null == e.key ? 0 : hash(e.key);
                }
                //计算新下标
                int i = indexFor(e.hash, newCapacity);
                //下面这两步很明显了，头插法。即先拿到桶原本指向的节点，把要插入的节点的next指向它，然后再把桶指向这个要插入的节点
                e.next = newTable[i];
                newTable[i] = e;
                //next记录了旧数组旧链表里的下一个节点，现在把e指向它，准备下次循环把它插到新桶
                e = next;
            }
        }
    }
```
可以看到 transfer 方法用的是头插法，我们重点注意以下几个步骤：
>1.Entry<K,V> next = e.next;     next指向旧链表中的下一个节点
>2.e.next = newTable[i];      把新链表的头结点摘下来放到将要插入的节点的尾部
>3.newTable[i] = e;        插入节点，即把桶指向e
>4.e = next;        把e指向next，准备继续插入下一个节点

# 多线程扩容
单线程的情况下扩容，是没什么问题的，我们直接来一步步分析多线程扩容的时候会出现什么问题。

>假如扩容前的数组和链表是这样的，数组里有2个桶，桶1里有三个节点，它们的键分别为3、7、5。

![扩容前](https://img-blog.csdnimg.cn/20201114210933754.png#pic_center)
>1 现在有两个线程同时扩容这个数组。假如线程一先拿到了cpu资源，执行完了 **Entry<K,V> next = e.next;** 这一步后，e指向3，next指向7。
>这时CPU资源交给了线程二，好家伙，线程二啪啦啪啦啪啦一下子就把整个链表都迁移了过去，直接扩容完成。
>这个时候旧数组、链表和两个线程的状态是这样的。
![线程二扩容完成](https://img-blog.csdnimg.cn/20201114211611593.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEwMTE1NQ==,size_16,color_FFFFFF,t_70#pic_center)

>2 现在轮到线程一继续执行了。线程一一顿操作猛如虎，执行完第一次循环后，把3插了进去，然后把next赋给了e，还记得在线程二执行的时候，线程一的next是7吗？现在e指向的节点就是7，准备下一次循环插入7。第一次循环过后变成了这样：
![第一次循环结束](https://img-blog.csdnimg.cn/20201114211819832.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEwMTE1NQ==,size_16,color_FFFFFF,t_70#pic_center)

>3 下次循环一开始，**Entry<K,V> next = e.next** 直接把next指向3。问题不大，没毛病。现在继续执行下去，把7的next指向3，然后再把桶指向7，即用头插法插入了7。执行完第二次循环后如下图，现在问题已经差不多出来了，执行完此次循环后，e指向了3，但是3已经被插入了，继续执行下去会怎么样呢？
![第二次循环结束](https://img-blog.csdnimg.cn/20201114212639171.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEwMTE1NQ==,size_16,color_FFFFFF,t_70#pic_center)

>4 这次循环执行情况是这样的
>Entry<K,V> next = e.next;    现在e是3，而3的next是null，所以next指向了null
>e.next = newTable[i];		这一步把3的next指向了7，因为桶现在是指向7的
>newTable[i] = e;			把桶指向3
>e = next;		由于next是null了，所以e指向了null，跳出了while进行后面的for循环
现在问题出来了，3的next指向了7，而7的next原本就指向3，情况如下图：
![第三次循环结束出大问题](https://img-blog.csdnimg.cn/20201114214042697.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEwMTE1NQ==,size_16,color_FFFFFF,t_70#pic_center)

显而易见，环形链表出现，这个时候如果我们再去做一些查询，查到这个桶3上面来的话，遍历的时候就出现了死循环，直接出不去了，出大问题……
还好JDK8修复了这个死循环问题。