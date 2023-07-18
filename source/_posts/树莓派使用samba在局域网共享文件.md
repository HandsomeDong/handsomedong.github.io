---
title: 树莓派使用samba在局域网共享文件
date: 2019-12-15 00:18:43
categories: 
    - 树莓派
---
新买的树莓派4终于到了！！！我的树莓派3B+光荣退休！到手第一步当然是用树莓派通过SMB共享我硬盘里的文件啦，随传随看。

![你懂的](https://s2.loli.net/2023/07/19/l72kpZNU8OXwPWy.jpg)

### 安装samba
samba是一个能让Linux系统应用Microsoft网络通讯协议的软件，而SMB是Server Message Block的缩写，即为服务器消息块 ，SMB主要是作为Microsoft的网络通讯协议，后来samba将SMB通信协议应用到了Linux系统上，就形成了现在的samba软件。后来微软又把 SMB 改名为 CIFS（Common Internet File System），即公共 Internet 文件系统，并且加入了许多新的功能，这样一来，使得samba具有了更强大的功能。

所以首先得安装samba。
```
apt-get install samba samba-common-bin
```

### 新建用户并与要共享的文件夹绑定

接下来先新建一个用户用于其它设备通过samba访问共享文件夹时登录访问。

```
    useradd share	//新建用户
    passwd share    //设置用户密码
    mkdir /home/share    //新建一个文件夹
    chown -R share /home/share    //绑定
```

![新建用户](https://s2.loli.net/2023/07/19/NtuRO2EzSZHdYyU.jpg)

### 配置samba

用户和文件夹都建好了，现在就要根据情况修改配置了。配置文件路径是 /etc/samba/smb.conf ，打开这个文件，在文件最后添加共享账户和共享文件夹的一些信息。

```
[HandsomeDong]	#网络中显示的文件名称
    path = /home/share	#文件路径
    valid users = share	#允许浏览的用户
    browseable = yes	#允许浏览
    public = yes		#允许共享访问
    writable = yes		#允许写入
```

### 设置共享密码以及重启

访问共享文件时，用的是samba设置的共享密码，而不是linux用户密码，现在要设置的就是共享密码，设置好后直接重启samba服务。

```
smbpasswd -a share
service smbd restart
```
![设置密码、重启samba服务](https://s2.loli.net/2023/07/19/ukJ68IjhBtnbegr.jpg)

### 访问共享文件夹
#### 电脑访问
用windows系统访问共享文件夹，只需在资源管理器输入 \\\\IP 就可以直接看到共享文件夹了，输入用户名和共享密码就可以访问了。现在把我前几天买的1T硬盘挂载到这个文件夹下，我就可以肆意地往里面存片子了！！！


![文件夹](https://s2.loli.net/2023/07/19/YdNgS6hDXkcOGp3.jpg)

![输入用户名、密码](https://s2.loli.net/2023/07/19/I2ERBc931Ylautb.jpg)

![片子](https://s2.loli.net/2023/07/19/UNiGLEF184wXqJW.jpg)

#### 安卓手机访问
安卓手机有不少APP都能使用SMB协议，我用的比较多的是 Solid Explorer 和 Kodi。Solid Explorer用来管理文件，视频、文本、音频、图片什么的浏览、管理起来都比较方便，Kodi浏览视频和图片非常好，非常强大！

### ipad、iphone访问
App Store上有很多软件都可以，收费的体验更好！

