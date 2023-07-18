---
layout: w
title: 除了 Websocket ，服务端还有什么办法能向浏览器主动推送信息？
date: 2021-11-13 13:43:47
tags: 
    - websocket
    - Server-Sent Events
categories:
    - 前端
---
# 前言
打工的时候，偶尔在闲暇时刻偷偷打开一下股票网站，看看今天有没有赚钱，说不定哪天一夜暴富，这是我现在唯一的盼头……

![流泪打工人](https://s2.loli.net/2023/07/19/euz1TniqLF2o7SI.jpg)

今天我一如往常打开熟悉的“XX财富网”，输入熟悉的股票代码，点开我前几天刚买入的“XXX”……

![绿](https://s2.loli.net/2023/07/19/7CPtpYo3VjMIRhy.gif)

看着冒着绿光并且一闪一闪的屏幕，我陷入了沉思……

![沉思](https://s2.loli.net/2023/07/19/K6dpJc7P23Z8iSD.jpg)

**我恍惚了一下，心想这实时刷新的数据，是后端通过websocket推过来的吗？**

于是我F12看了一下，**没有 websocket 连接啊！**

![没有websocket](https://s2.loli.net/2023/07/19/7NDhloYLMPnKciZ.png)

**难道除了 Websocket 还有别的办法能够让后端向前端主动推送消息？**

我马上想到了 http2.0 的 server push ，但是想了想又觉得不太可能，因为 http2.0 的push并不是这个意思。

**http2.0 的 server push 指的是：服务器还没有收到浏览器的请求，服务器就把各种资源推送给浏览器。比如，浏览器只请求了 index.html，但是服务器发现 index.html 内容里面需要用到 style.css、example.png，于是服务器在浏览器请求 index.html时顺便把 style.css、example.png 也一起发送给浏览器。这样的话，只需要一轮 HTTP 通信，浏览器就得到了全部资源，提高了性能。**

所以很明显，http2.0 的 server push 只是复用其中一条http连接顺便把后续需要发送的一些静态资源提前发给了浏览器，并不**适用于这种推送动态数据的场景**（比如股票涨跌数据实时刷新、直播弹幕推送等）。

那服务器究竟是怎么实时推送了股票相关的数据呢？
# 端倪
于是我点开了 Fetch/XHR 看看到底是何方神圣，在不断推送数据。于是我发现了这几个请求的内容好像不断地在刷新……

从下图可以看到一秒钟推送好几条数据过来（现在是十一点半，我打开了美股）

![端倪](https://s2.loli.net/2023/07/19/9ROFre7IysTmwzv.png)

查看这几个http请求，很容易就发现请求头和响应头的 Content-Type 和 Accept 跟其它普通的http请求不太一样。

![头](https://s2.loli.net/2023/07/19/U1qVtv7mO6McDiH.png)

很明显，这个服务端推送，跟这个 text/event-stream 关系很大。

![真相只有一个](https://s2.loli.net/2023/07/19/DE4y1Q5nNtKHsOI.gif)

于是我去查了一下相关资料，果不其然……**服务器向浏览器推送信息，除了 WebSocket，还有一种方法：Server-Sent Events。**

# Server-Sent Events 是什么？
Server-Sent Events，服务器发送事件，简称SSE，是一种 HTML 5 事件通知，它允许网页获得来自服务器的更新。

严格来说，HTTP 协议是没有办法做到服务器主动推送信息的，但是有一种变通的方法可以做到，那就是服务器向客户端声明接下来要发送的是流信息（streaming）。

也就是说，发送的不是一次性的数据包，而是一个数据流，会连续不断地发送过来。这时，客户端不会关闭连接，会一直等着服务器发过来的新的数据流，视频播放就是这样的例子。本质上，这种通信就是以流信息的方式，完成一次用时很长的下载。

SSE 就是利用这种机制，使用流信息向浏览器推送信息。它基于 HTTP 协议，目前除了 IE/Edge，其他浏览器都支持。

# Server-Sent Events 与 Websocket 对比
既然能让服务器主动向浏览器推送信息，那我们肯定会想到让websocket来跟它做做对比。

总的来说，Websocket 应该是更强大、更灵活、应用更广泛的，因为它是全双工通信，可以双向通信；但是SSE是单向通道，只能服务器向浏览器发送，因为流信息本质上就是下载。如果浏览器向服务器发送信息，就变成了另一次 HTTP 请求。
![SSE](https://s2.loli.net/2023/07/19/9QgO1zIvSjAcw7F.png)
但是，SSE也有自己的优点。
* SSE 使用 HTTP 协议，现有的服务器软件都支持。WebSocket 是一个独立协议。
* SSE 比较轻量级，使用较简单；WebSocket 协议相对来说比较复杂。
* SSE 默认支持断线重连，而WebSocket 需要自己实现。
* SSE 一般只用来传送文本，二进制数据需要编码后传送，WebSocket 默认支持传送二进制数据。
* SSE 支持自定义发送的消息类型。

所以 SSE 和 Websocket 适合不同的使用场景。

迟点有空的时候写个demo试试。

# 总结
* 除了 Websocket ，服务端还能通过 SSE 向浏览器主动推送消息，其本质上就是一个基于 HTTP 的流。
* 今天的工不白打，股市虽绿，但我因此收获了一个冷门的知识点~~安慰自己~~ 。

![我真的没哭](https://s2.loli.net/2023/07/19/NAxtRGfvVPlozEe.jpg)