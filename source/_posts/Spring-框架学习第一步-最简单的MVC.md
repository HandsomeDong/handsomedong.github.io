---
title: 'Spring 学习—第一个MVC：Hello World'
date: 2019-04-20 04:14:29
tags: 
categories: 
    - Java
    - 框架
    - Spring
---
# Spring 学习—第一个MVC：Hello World
Spring框架是Java应用最广的框架，它是开源的，是为了解决企业应用程序开发复杂性而创建的，想要做Java Web开发，怎么能不会Spring呢？想要学Spring，那第一步先学习怎么用Spring框架创建一个最简单的MVC吧！笔者用的是IDEA，所以这里就用IDEA来创建。
## 1.创建项目
首先当然是在IDEA新建项目。

![IDEA新建项目](https://i.loli.net/2019/04/22/5cbc9d90cc126.png)
类库选择"Spring MVC"，并且在JAVA EE中选择"Web Application"。

![选择类库](https://i.loli.net/2019/04/22/5cbc9f6326dd4.png)
输入项目名称以及选择项目的地址，然后点击finish。

![输入项目名](https://i.loli.net/2019/04/22/5cbca08ee6363.png)
点击finish后，IDEA会自动下载需要的类库，大概等几分钟。

![下载类库](https://i.loli.net/2019/04/22/5cbca08ec5768.png)
下载类库完毕后，项目就建立完成了！

![项目文件](https://i.loli.net/2019/04/22/5cbca1a33bcbd.png)

---
## 2.调试设置
项目建立好后，可以看到右上角的run和debug按钮都是灰色的，不能运行调试，所以接下来我们要在IDEA进行运行、调试的相关设置。

![灰色的运行和调试按钮](https://i.loli.net/2019/04/22/5cbca7fa00965.png)
在菜单栏中找到 Run->Edit Configurations，可以打开运行设置框。

![运行调试配置](https://i.loli.net/2019/04/22/5cbca82fb4317.png)
按照下图添加本地Tomcat服务器配置。

![添加服务器](https://i.loli.net/2019/04/22/5cbca857a68f6.png)
Name就是该配置的名称，随便取；After launch是设置你运行项目所自动打开的浏览器以及打开的URL，我喜欢用谷歌；HTTP port就是该项目的运行端口。

![输入相关配置](https://i.loli.net/2019/04/22/5cbca873eec91.png)
然后再点击Deployment-> + ->Artifacy，设置好部署的url，直接填个"/"就行了，当然你也可以加点东西，比如填"/handsomedong"然后点击 OK 。

![添加Artifacy](https://i.loli.net/2019/04/22/5cbca89037029.png)

![应用目录](https://i.loli.net/2019/04/22/5cbca8abc0f47.png)
设置好运行调试的配置后，我们就可以在右上角看到配置名称了，同时也可以看到有个三角符号和虫子都变绿了。三角符号是运行按钮，点击后直接运行，小虫子是调试按钮，可以设置断点调试。

![变绿了](https://i.loli.net/2019/04/22/5cbca9046f750.png)
我们先试试运行一下会怎么样吧！结果不出意料，根本没跑起来……我们在IDEA下面查看日志，可以看到报错信息
<font color=#FF0000><center>*java.lang.ClassNotFoundException: org.springframework.web.context.ContextLoaderListener* </center> </font>
意思是找不到这个类！为什么呢？因为还有一些包没有jar包含进来。

![日志报错信息](https://i.loli.net/2019/04/22/5cbca9202eeaa.png)

---

## 3.导入相关类库
点击左上角 File->Project Structure。

![Project Structure](https://i.loli.net/2019/04/22/5cbcac519ce36.png)
点击Artifacts再点击项目后，可以看到错误的提醒，缺少了一些引用。我们可以通过点击"Fix"来查看IDEA有什么解决办法。

![Artifacts](https://i.loli.net/2019/04/22/5cbcac51ab62d.png)
可以看到IDEA能为我们加入这些类库，那我们就点击最上面两个吧(请忽略我的QQ拼音输入法头像)！

![添加 Spring MVC-1.3.18 RELEASE](https://i.loli.net/2019/04/22/5cbcad4fbae7d.png)
添加完这些类库后，就可以看到已经没有错误的提醒了，我们再重新运行项目看看如何吧！

![重新运行](https://i.loli.net/2019/04/22/5cbcac5171a78.png)
这次终于运行成功了！！！运行成功后，IDEA会自动使用我们配置的浏览器，打开我们配置的URL！看看此时的运行结果，其实就是 web目录下的 index.jsp文件。

![运行结果](https://i.loli.net/2019/04/22/5cbcac51642d0.png)

![index.jsp文件](https://i.loli.net/2019/04/22/5cbcac517e20f.png)

---

## 4.添加控制器
既然项目能跑了，那我们就得正式开始写MVC了，我们先写一个Controller吧！
我们首先要添加一个新的包，报名自己取，我在src目录添加了一个名为"com.handsomedong.springmvc"的包。

![添加新包](https://i.loli.net/2019/04/22/5cbcb22d90dd9.png)

![输入包名](https://i.loli.net/2019/04/22/5cbcb22d7f475.png)
然后在这个包下添加一个Java Class，这个class就是控制器了，取名叫HelloController吧！

![添加Class](https://i.loli.net/2019/04/22/5cbcb22d8adf5.png)

![输入类名](https://i.loli.net/2019/04/22/5cbcb22d67bfd.png)
HelloController代码如下。
在类上面添加注解 *@Controller* 注明该类是个控制器、*@RequestMapping("/hello")* 注明该控制器的路由为"/hello"，在方法test()上面添加注解 *@RequestMapping("world")* 注明该方法的路由为"world"。

```
package com.handsomedong.springmvc;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
@RequestMapping("/hello")
public class HelloController {

    @RequestMapping("world")
    public String test()
    {
        return "/WEB-INF/jsp/world.jsp";
    }
}
```

因此我们想要访问该方法时，url应该为"/hello/world.form"，我知道你一定要问我为什么会有".form"，那是因为默认配置就是这样！

我知道你肯定又会问我能不能在配置中把".form"去掉，当然能！我们可以在 /web/WEB-INF/dispatcher-servlet.xml 中找到这么一段代码。把其中的 \*.form 改成 / 就行了

```
    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <url-pattern>*.form</url-pattern>
    </servlet-mapping>
```

## 5.添加视图
其实 HelloController 的 test() 意思是返回 /WEB-INF/jsp/world.jsp 这个View，所以我们要在 WEB-INF 下新建文件夹 jsp ，并且新建一个jsp文件。world.jsp 代码如下。

```
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>HandsomeDong</title>
</head>
    Hello, world.
<body>

</body>
</html>
```

此时控制器和视图都建好了，我们运行一下项目，在浏览器输入 http://localhost:8080/hello/world 看看效果如何吧！

![被404支配的恐惧](https://i.loli.net/2019/04/22/5cbcb9a734bef.png)
怎么样，是不是要枯了……怎么又是404！！！年轻人别太心急，想想程序跑不起来我们第一步该干什么。

![bug？一定是电脑的问题](http://tc.sinaimg.cn/maxwidth.2048/tc.service.weibo.com/p/www_apkbus_com/d4e669d65a97f2ee45af6bc42ba016ff.gif)
年轻人还是要戒骄戒躁啊，我们先来看看服务器这边报错好吧。

![报错信息](https://i.loli.net/2019/04/22/5cbcb9a73e678.png)
这里说 DispatcherServlet 没有找到响应 /hello/world 的控制器！为什么呢？我们路由配好了，输入的URL也没有错啊！
原因是我们还要告诉Spring去哪里找这些控制器……


我们找到 /web/WEB-INF/dispatcher-servlet.xml 这个文件，在 beans 里面添加这么一行代码，意思就是你记得要去我这个包里扫描控制器！！！

```
<context:component-scan base-package="com.handsomedong.springmvc"/>
```
此时我们再重新运行一下试试看吧！终于成功了！！！

![Hello, world.](https://i.loli.net/2019/04/22/5cbcbb344abac.png)

---
## 6.添加模型
现在Controller和View都已经完成了，就剩一个Model了，我们写个最简单的吧，直接在Controller传入一个Model。回到HelloController，修改代码如下。
```
package com.handsomedong.springmvc;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
@RequestMapping("/hello")
public class HelloController {

    @RequestMapping("world")
    public String test(Model model)
    {
        model.addAttribute("name", "HandsomeDong");
        model.addAttribute("github", "https://github.com/handsomedong");
        return "/WEB-INF/jsp/world.jsp";
    }
}
```

再把Model的值放到页面中显示出来，现在把 world.jsp 代码修改如下。
```
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>HandsomeDong</title>
</head>
<body>
    <p>Hello, ${name}.</p>
    <a href="${github}"> GitHub </a>
</body>
</html>
```
现在运行查看结果吧！

![运行结果](https://i.loli.net/2019/04/22/5cbcbe1973bcb.png)
至此我们就完成了Spring框架最简单的MVC项目了！


