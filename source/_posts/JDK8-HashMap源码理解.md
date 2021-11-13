---
title: JDK8 HashMap源码理解
date: 2020-11-14 18:09:51
tags:
    - HashMap
categories: 
    - Java
    - 基础
---
# 前言

<font color=#999AAA >最近本翔再次认真阅读了HashMap的源码，总结一下自己的一些理解，希望对大家能有点帮助。

# 一、数据结构
HashMap的数据结构是数组+链表，数组里面是一个个Node，在JDK8中加入了红黑树。HashMap根据存入的对象的hash跟数组长度取模得出下标，如果发生了哈希冲突，则新插入的键值对会放到上一个节点的后面，形成链表。当链表长度超过8且数组长度也不小于64时会转为红黑树，同理，链表长度小于6时会再次变回链表。数组里的每个存储空间，我们称为桶(bucket)。
![HashMap内部](https://img-blog.csdnimg.cn/20201114005720673.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEwMTE1NQ==,size_16,color_FFFFFF,t_70#pic_center)
## 1.1 Node
Node封装了key和value，我们使用put方法塞进去key和value其实都存到了Node里面，看看Node里面的成员变量有哪些。
```c
    static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        Node<K,V> next;
        // ………………略
    }
```
可以看到里面有四个成员
>hash值：即键的散列值
>key：就是我们put进去的那个键了
>value：就是我们put进去的那个值了
>next：下一个节点

## 1.2 重要参数
HashMap里面定义了一些比较重要的参数
```c
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;    //初始容量
static final int MAXIMUM_CAPACITY = 1 << 30;    //最大容量
static final float DEFAULT_LOAD_FACTOR = 0.75f;    //负载因子，当map里面的node数量大于 threshold = loadFactor * capacity时，就会扩容
static final int TREEIFY_THRESHOLD = 8;    //桶的树化阈值，转为红黑树的哈希冲突数量阈值
static final int UNTREEIFY_THRESHOLD = 6;    //桶的链表还原阈值，转为链表的哈希冲突数量阈值
static final int MIN_TREEIFY_CAPACITY = 64;    //最小树形化容量阈值，当哈希表中的容量即数组长度不小于该值时，树形化链表
```
>1.初始容量和加载因子我们在实例化HashMap的时候是可以定义的。当 元素数量 > 容量*加载因子 时，数组就会扩容。
>2.当桶里面的 链表长度 > TREEIFY_THRESHOLD 且 桶数组长度 > MIN_TREEIFY_CAPACITY 时，链表就会转换为红黑树。

# 二、源码分析

## 2.1 扰动函数
没仔细看源码前，我一直以为计算桶下标的时候值直接用键的hashCode跟数组容量相与得出的下标，现在仔细一看才发现事情没那么简单。
```c
    static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
```
可以看到扰动函数里面把hashCode右移了16位再与原hashCode进行了异或操作，即用自己的高16位和低16位进行了异或。这样能增加随机性，让数据元素更散列，减少碰撞。

## 2.2 插入
简单来说，插入键值对的时候，先通过键的哈希值计算出下标，然后再把数据存放到数组中。不多BB，直接放源码。

```c
    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }
    
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        //如果数组未初始化，则先初始化
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        //如果桶里没数据，直接新建一个Node，并把桶指向这个Node
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        //桶里有数据再做处理
        else {
            Node<K,V> e; K k;
            //键值和键散列值都和桶里的第一个Node键值和键hashCode一样，直接把e指向这个Node，准备后面直接替换值
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            //如果是TreeNode对象，则调用红黑树插入值的方法
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            //即不是树节点，且第一个Node也不是要找的Node，那就准备遍历链表
            else {
            	//遍历链表且统计链表长度
                for (int binCount = 0; ; ++binCount) {
                	//遍历完链表都没有这个key，那就在链表后面插入新节点吧
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        //链表长度大于阈值的话就转为红黑树
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    //找到key，结束遍历
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            //e不为空则表示链表里包含有要插入的键值对，直接替换值就好了
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        //元素数量超过threshold时就扩容
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```
主要步骤如下：
>1.检查桶数组是否为空，为空则resize()初始化
>2.计算下标，用桶数组长度 - 1跟键的哈希值相与，即(length - 1) & hash 计算下标，看桶是否为空，为空就直接把Node存进桶中
>3.如果桶里第一个Node的key和当前的key一样，直接将桶指向现在的Node
>4.如果Node是树节点，就调用红黑树的插入方法
>5.如果以上两种情况都不是，则需要遍历该链表，找到对应的key替换值或者没有key就往链表尾部插入新Node
>6.所有元素处理完成后，判断size是否超过阈值 threshold，是则resize()扩容

## 2.3 查找
查找就比较简单，就计算出数组下标，到桶里面看看是红黑树还是链表还是null，再拿数据。
```c
    public V get(Object key) {
        Node<K,V> e;
        return (e = getNode(hash(key), key)) == null ? null : e.value;
    }

    final Node<K,V> getNode(int hash, Object key) {
        Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
        //桶里有数据
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (first = tab[(n - 1) & hash]) != null) {
            //如果桶里第一个就是要找的，就直接返回
            if (first.hash == hash && // always check first node
                ((k = first.key) == key || (key != null && key.equals(k))))
                return first;
            if ((e = first.next) != null) {
            	//树节点，调红黑树的查询方法
                if (first instanceof TreeNode)
                    return ((TreeNode<K,V>)first).getTreeNode(hash, key);
                //遍历链表
                do {
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        return e;
                } while ((e = e.next) != null);
            }
        }
        return null;
    }
```
## 2.4 树形化
由于从链表中查找数据的时间复杂度是O(n)，所以如果插入的键值对，发生哈希碰撞太多而导致链表过长的时候，那性能是非常差的。所以在JDK8中，链表长度大于8且桶容量不小于64时，链表会树形化，即转为红黑树。

```c
    final void treeifyBin(Node<K,V>[] tab, int hash) {
        int n, index; Node<K,V> e;
        //可以看到，如果桶数组容量小于64时，就算链表长度大于8，也只会扩容而不会把链表转为红黑树
        if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
            resize();
        else if ((e = tab[index = (n - 1) & hash]) != null) {
            TreeNode<K,V> hd = null, tl = null;
            do {
            	//注意，这里只是把普通节点转成了树节点，现在还不是红黑树
                TreeNode<K,V> p = replacementTreeNode(e, null);
                if (tl == null)
                    hd = p;
                else {
                    p.prev = tl;
                    tl.next = p;
                }
                tl = p;
            } while ((e = e.next) != null);
            if ((tab[index] = hd) != null)
            	//这里才是转红黑树，红黑树内容略多，就不在这篇博客详细总结了
                hd.treeify(tab);
        }
    }
```

## 2.5 扩容
扩容可以有效避免有太多的哈希碰撞而导致的桶内数据过多的情况。桶数组容量 * 负载因子大于元素数量时，HashMap就会扩容，把数组容量扩大一倍。
```c
    final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold;
        int newCap, newThr = 0;
        //容量不为空，即已经初始化
        if (oldCap > 0) {
        	//容量达到 2^30 ，不会再扩容
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            //容量增加一倍
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; // double threshold
        }
        else if (oldThr > 0) // initial capacity was placed in threshold
            newCap = oldThr;
        else {               // zero initial threshold signifies using defaults
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        threshold = newThr;
        //初始化新的桶数组
        @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
        if (oldTab != null) {
        	//遍历旧的桶数组
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                //桶里面有数据，准备迁移
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null;
                    //如果桶里只有一个节点的话，直接把新桶指向这个节点
                    if (e.next == null)
                        newTab[e.hash & (newCap - 1)] = e;
                    //如果桶里指向的是树节点，则调用红黑树的拆分操作
                    else if (e instanceof TreeNode)
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    //桶里的链表不止一个节点，则需要遍历链表一个个迁移
                    else { // preserve order
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {
                            next = e.next;
                            //这里用哈希跟旧数组长度相与，计算出该节点是放在高位桶还是低位桶
                            if ((e.hash & oldCap) == 0) {
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
                            else {
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                        //把新桶指向链表头节点，数据迁移完成
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        if (hiTail != null) {
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
        return newTab;
    }
```
总结扩容操作步骤，其实主要就以下几个步骤：
>1.没初始化就初始化，容量达到MAXIMUM_CAPACITY即2^30就不扩容了
>2.新建桶数组，容量为旧的桶数组的2倍
>3.遍历旧数组的所有桶，桶里链表只有一个数据就直接迁移，是树节点就调用红黑树的拆分方法，链表不止一个数据就遍历链表
>4.由于新桶数组容量是原来的两倍，所以原本在同一个桶中的数据迁移到新数组中时，是有可能放在不同的两个桶中的。一个是低位桶，下标值和原来的一样，一个是高位桶，下标值和原来的差值为旧桶数组的容量，通过 hash&oldCap 是否等于0可以判断这个节点是在高位桶还是低位桶

为什么 (hash&oldCap==0) 可以判断一个节点是在高位桶还是低位桶呢？
>一个节点的数组下标，是通过它的键的hash跟数组容量取模计算得来的，所以同一个桶上的数据，要么它们的hash相等，要么它们的hash相差 2^n 个数组容量。
>比如两个哈希值分别为13和5的键，插进数组长度为8的桶数组里，1101&0111 和 0101&0111 计算得到的下标都是0101，即5。当扩容后，数组长度为16，而这时再计算下标时， 就变成了 1101&1111和0101&1111，最终的结果分别为13和5，它们相差8。哈希值为5的节点扩容后仍然在下标值为5的桶上，而哈希值为13的节点已经迁移到了下标值为8的桶上了。
>其实不难发现，通过位于运算计算下标时，扩容前的0111和扩容后的1111只相差第四位的那个1，由于哈希值为13的第四位原本是1，而哈希值为5的第四位原本是0，当位于运算的值有0111变成1111时，第四位也参与了运算，才导致了扩容后的它们得出的下标值会不同。所以我们可以直接用1000跟hash进行相与，看看它的第四位是1还是0就可以知道它是在高位桶还是低位桶了。

# 三、常见问题
## 3.1 桶数组长度为什么总是2的n次方
>计算效率更高，计算更方便。
>取模(%)操作中如果除数是2的幂次，则等价于与其除数减一的与(&)操作（前提是length是2的n次方）。所以可以从源码中看到，计算数组下标是通过 hash & (length - 1) 来计算的。而且这样使扩容后计算新位置会更方便，从扩容源码中可以看到判断一个节点扩容后的数组下标是在高位桶还是低位桶时，直接用hash跟旧桶数组容量相与即可判断，并在如果是高位桶，直接用下标加上旧数组长度即可拿到高位桶的下标。

## 3.2 HashMap的参数loadFactor，它的作用是什么？
>其实从源码中可以看得很清楚了，它其实是表示HashMap里面数组所能承受的拥挤程度，影响hash操作到同一个数组位置的概率。默认的loadFactor是0.75，当HashMap里面容纳的元素已经达到HashMap数组长度的75%时，表示HashMap太挤了，需要扩容，在HashMap的构造器中可以定制loadFactor。

## 3.3 HashMap从JDK7到JDK8的变化。
>1.树形化
>首先肯定就是新增了红黑树啦，JDK7及之前桶中只有链表一种数据结构，JDK8中桶内元素大于8且数组容量不小于64时便会把桶内链表转为红黑树。
>2.hash计算方式不同
>JDK7的扰动函数，前前后后进行了四次扰动。而JDK8简化了这一操作，只是对高低位做了异或。
>3.扩容操作中，下标的计算方法不一样。
>可以从JDK8源码中看到resize()里面，判断节点是高位桶还是低位桶，是高位桶的话，直接用旧的下标加上旧的数组容量即可。但是JDK7中不是这样的，JDK7的扩容操作需要对节点重新计算下标。
>4.发生哈希碰撞时，插入链表中的方法不一样。
>JDK8里的HashMap插入元素时，用的是尾插法，但是JDK7里用的是头插法。在进行扩容操作迁移元素时也是如此，这就导致JDK7的HashMap在多线程进行扩容操作时，有可能会出现死锁问题！这里就不详细描述JDK7HashMap的死锁问题了，后面会专门出篇博客翔谈该问题。（不要问我为什么是“翔谈”不是“详谈”，我叫七里翔）