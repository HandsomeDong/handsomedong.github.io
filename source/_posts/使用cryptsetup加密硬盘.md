---
title: 使用cryptsetup加密硬盘
date: 2019-12-09 21:54:35
categories: 
    - 树莓派
---
在京东关注移动硬盘好久了，等了很久终于等到希捷移动硬盘几个月以来的最低价……299！咬咬牙终于下单，今天早上到手。


![1T希捷移动硬盘](https://img-blog.csdnimg.cn/20201114185225708.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEwMTE1NQ==,size_16,color_FFFFFF,t_70#pic_center)


这个移动硬盘主要用来给我的树莓派外接，然后用SMB共享，这样在内网条件下，我的手机、电脑、平板等就可以随时访问硬盘里的内容了，不过在此之前，我还得对硬盘进行加密，免得谁偷偷把我硬盘拿走了后随意浏览里面一些不可描述的东西。
cryptsetup是linux下的一个分区加密工具，它通过调用内核中的"dm-crypt"来实现磁盘加密的功能，这个加密工具应该还挺常用的，我的树莓派就是用的这个加密工具。不过在此之前，我们得先初始化这个硬盘

![不可描述](https://img-blog.csdnimg.cn/20201114185314876.jpg#pic_center)


### 分区

#### 找到设备
第一步先对硬盘进行分区。把移动硬盘插到树莓派上，用 **fdisk -l** 看看是哪个硬盘。

![fdisk -l](https://img-blog.csdnimg.cn/2020111418540061.jpg#pic_center)

很明显这个 /dev/sdb1 就是我插进去的移动硬盘。

#### 创建分区

现在输入命令 **fdisk /dev/sdb1** 来操作这个硬盘。然后可以输入 m 来查看操作提示。

![操作提示](https://img-blog.csdnimg.cn/20201114185454794.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEwMTE1NQ==,size_16,color_FFFFFF,t_70#pic_center)


可以看到 n 是添加新分区，那就输入 **n** 吧。
![添加新分区](https://img-blog.csdnimg.cn/20201114185659938.jpg#pic_center)

这个时候询问我们是要创建主分区还是扩展分区，主分区最多只能创建4个，如果创建了扩展分区那么扩展分区需要占用一个主分区，直接enter则默认是主分区，后面还有询问分区序号、分区大小什么的，我一路直接enter了，即默认。
![创建主分区](https://img-blog.csdnimg.cn/20201114185740894.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEwMTE1NQ==,size_16,color_FFFFFF,t_70#pic_center)


创建好后可以输入 **p** 来查看逻辑分区。
![查看逻辑分区](https://img-blog.csdnimg.cn/20201114185821330.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEwMTE1NQ==,size_16,color_FFFFFF,t_70#pic_center)

可以看到我此时是只有一个分区的，大小是931.5G，现在可还没分区成功，还需要输入 **w** 保存。

### 加密格式化
OK，现在硬盘基本的初始化完成了，现在要开始对它进行加密了。我这里用的是分区加密工具是cryptsetup，没有的话需要先安装。安装方法很简单，这里就不详细介绍了。
现在输入 **cryptsetup luksFormat /dev/sdb1** ，然后输入YES确认（注意要大写），再输入你要给硬盘设置的密码。
![加密格式化](https://img-blog.csdnimg.cn/20201114185918841.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEwMTE1NQ==,size_16,color_FFFFFF,t_70#pic_center)

### 打开并挂载硬盘
#### 打开加密的硬盘

输入 **cryptsetup luksOpen /dev/sdb1 test** ，并且输入密码后，硬盘 /dev/sdb1 会被映射到 /dev/mapper/test 下。
![打开硬盘](https://img-blog.csdnimg.cn/20201114185954951.jpg#pic_center)


#### 格式化映射的设备
现在还需要再格式化一下映射的设备，执行 **mkfs.ext4 /dev/mapper/test**，这步操作只需在第一次打开设备时需要，以后再打开、挂载硬盘可以就无需这步操作了。

![格式化映射设备](https://img-blog.csdnimg.cn/20201114190037490.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEwMTE1NQ==,size_16,color_FFFFFF,t_70#pic_center)

#### 挂载硬盘
这个时候想要挂载硬盘，直接把映射的目录挂载就行， **mount /dev/mapper/test /home/HandsomeDong** 。
![挂载硬盘](https://img-blog.csdnimg.cn/20201114190121843.jpg#pic_center)


### 卸载硬盘

想要卸载、拔出硬盘，需要先卸载挂载点，再关闭映射的设备，切忌热插拔，容易损坏硬盘。
卸载挂载点 **umount /home/HandsomeDong**
关闭映射设备 **cryptsetup luksClose test**
![卸载硬盘](https://img-blog.csdnimg.cn/20201114190154672.jpg#pic_center)
### 可能遇到的问题
#### 数据无法读取
硬盘意外断电再重新挂载时可能会造成数据无法读取，这时可以使用fsck修复指定分区。
**fsck /dev/mapper/HandsomeDong**
![修复分区](https://img-blog.csdnimg.cn/20201114190246843.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEwMTE1NQ==,size_16,color_FFFFFF,t_70#pic_center)



