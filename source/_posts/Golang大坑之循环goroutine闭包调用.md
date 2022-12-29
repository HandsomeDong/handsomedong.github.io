---
title: Golang大坑之循环goroutine闭包调用
date: 2022-12-29 23:40:35
tags:
    - 高并发编程
categories: 
    - Golang
---

# 前言
回顾整个2022，突然发现我一篇博客都没写，趁着还没2022还没过去，赶紧~~水一篇博客~~ 分享一下我最近学习到的一些东西，这次的主题是“Golang大坑之循环goroutine闭包调用”，大家就当小故事来看吧。
![躺好，听我讲故事](https://img-blog.csdnimg.cn/7de6784a7c274a23bc4d2af416690dcd.jpeg)
# 小美又写了bug
仔细看，这个女孩叫小美，她最近刚入职某Go大厂，从Java转了Go，小美入职仅仅花了半天就学完了Go语法，然后三天写了5个bug，隔三差五就被leader小帅叫到会议室臭骂。
![小美](https://img-blog.csdnimg.cn/03a694e0a4a34b67a48d4167e2d367d7.gif)
这不，正当小美写bug写得起劲，小帅又过来拍了拍小美肩膀：“小美，来下会议室”。
![小美来下会议室](https://img-blog.csdnimg.cn/5bfb8c0e0be2435db8e944ef20b08c76.png)
一来到会议室，小帅直接把笔记本拍到小美脸上：“看看你干的好事。”
笔记本上面是小美的提交记录，上面有这么一段代码。

```go
func main() {
	nums := []int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
	for _, num := range nums {
		go func() {
			fmt.Printf("打工人%d号\n", num)
		}()
	}
	time.Sleep(time.Microsecond * 1000)
}

```

小美看完表示很委屈：“这有什么毛病吗？这里一共开了十个协程，每个协程打印的编号都不一样，这里的输出应该是下面这样啊？出什么问题了吗？”
小美预期的结果：
```
打工人9号
打工人2号
打工人3号
打工人0号
打工人1号
打工人5号
打工人7号
打工人8号
打工人4号
打工人6号
```

小帅一巴掌甩小美脸上：“要不你看看运行结果？”
![一巴掌](https://img-blog.csdnimg.cn/4a23293bb3e64f0d97667839e97111a2.png)
小美运行了一下，结果让她大吃一斤翔。
![运行结果](https://img-blog.csdnimg.cn/2702364608904d238939e55e2abdca3b.png)
面对这个结果，坚强的小美也落下了泪，她实在不明白为什么会这样。
![落泪的小美](https://img-blog.csdnimg.cn/50fbd139792744d39465d5cf14d6f79d.jpeg)
# 小翔看出问题所在
谁也没有注意到，此时坐在会议室角落旁的小翔缓缓站了起来：“真相只有一个！”
![小翔登场](https://img-blog.csdnimg.cn/7b5b28e1cf4442949cdeb5571a466b15.png)
小翔继续说：“这是Golang新手及其容易犯的一个错误！**你这里的goroutine执行了闭包内的程序，而闭包内直接引用了num变量，这个值并没有被保存到goroutine栈中，这样写会导致for循环结束后才执行goroutine多线程操作，所以你看，很多个协程打印的编号都是9，即for循环结束后协程才运行，这个时候num值其实指向了最后一个元素。当然，有个别协程打印的不是9，因为它们运行得比较早。** 这样写及其容易窜数据，产生严重bug！”

**小帅一巴掌甩小翔脸上：“还TM装逼？快给我修bug。”**
![逼都让你装完了](https://img-blog.csdnimg.cn/45f5b499c86548cda3bd5de6f0948e22.jpeg)
# 小翔的解决方案
小翔不愧是小翔，是见过各种大场面的男人，只见他不慌不忙给出了几种解决方案。

## 正确写法1：不使用闭包
既然闭包有问题，那我们不使用闭包不就行了？多大点事啊？
```go
func main() {
	nums := []int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
	for _, num := range nums {
		go print(num)
	}
	time.Sleep(time.Microsecond * 1000)
}

func print(num int) {
	fmt.Printf("打工人%d号\n", num)
}
```

## 正确写法2：循环内定义新变量
由于循环内定义的变量在循环遍历过程中是不共享的，所以我们可以在循环内再定义新变量。
```go
func main() {
	nums := []int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
	for _, num := range nums {
		num := num
		go func() {
			fmt.Printf("打工人%d号\n", num)
		}()
	}
	time.Sleep(time.Microsecond * 1000)
}
```

## 正确写法3：闭包传参（优雅，推荐）
使用闭包时，我们还是尽量使用闭包传参，避免直接在闭包内使用外部的变量，比较容易出错。我个人认为这样写起来会比较优雅，推荐这种写法。
```go
func main() {
	nums := []int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
	for _, num := range nums {
		go func(num int) {
			fmt.Printf("打工人%d号\n", num)
		}(num)
	}
	time.Sleep(time.Microsecond * 1000)
}
```

## 运行结果
上面3种写法都会得到正确的结果
![运行结果](https://img-blog.csdnimg.cn/a3894ce33b9b44229ae933e0334eea86.png)
# 小翔的静态扫描工具
## 怎么从根源上避免这个问题
解决完了这个bug之后，leader小帅的脸色也有所好转，然后问了问小翔：“既然新手都比较容易犯这个错误，那我们有没有什么办法或手段来尽量让后面入职的同学少犯甚至不犯这个错误呢？换句话说，**怎么从根源上避免出现这个问题**？”
![怎么从根源上避免这个问题](https://img-blog.csdnimg.cn/eac696749fcb4d2cac1bcd340cfc2d87.png)
小翔推了推眼镜，冷静回答：“这个简单，回头我出个《Golang大坑》系列文档，让入职转Golang的新同学都看一下这个文档，他们就不会犯这个错误了！”
![小帅推眼镜](https://img-blog.csdnimg.cn/419e06197dbe45b9b59a934a49ed72dd.jpeg)
小帅直接就是一巴掌：“其实，我对你是有点失望的。你这个层级，不是把事情做好就可以的。你需要有体系化思考的能力。你做的事情，他的价值点在哪里？你是否作出了壁垒，形成了核心竞争力？你做的事情，和公司内其他团队的差异化在哪里？你的事情，是否沉淀了一套可复用的物理资料和方法论？为什么是你来做，其他人不能做吗？你需要有自己的判断力，而不是我说什么你就做什么…………（省略一千字）**你确定你写个文档大家就会去看，就能遵守这一套约定？**”
![小帅装逼](https://img-blog.csdnimg.cn/4ccfe314ffad49c08228ce52cfb2f52d.jpeg)
## 静态扫描工具
小翔这次终于是忍不住了，直接甩出了他的终极大招 -- 静态扫描工具，缓缓说道：**“这个是我写的Golang静态扫描工具，名叫 **xianggolint**，我刚刚加上了一个新规则：loopgoroutinecheck，它能扫描出循环内goroutine闭包直接引用循环index、value、key的代码，这种代码都是有问题的，Github 链接点[这里](https://github.com/HandsomeDong/xianggolint)，具体的使用方法可以看 readme 文档，以下是测试的扫描结果，可以看到扫描结果显示某个文件的第 12 行代码是有问题的，提示信息非常清晰。如果我们的CI/CD做得比较完善的话，我们还能将这种静态扫描工具接入到上面，在MR的时候进行扫描，扫出问题时就block进程，并提示该问题，这样我们就能在代码合入的阶段防止问题代码合入主分支啦！**”
![代码](https://img-blog.csdnimg.cn/a416464dfb814b1e90cac66e895af79e.png)
![代码](https://img-blog.csdnimg.cn/f00597bc51604d17a2bd1eb0ce7e2770.png)
小翔此时心里默默说：“今天这个装逼王，我当定了！”
![小翔装逼王](https://img-blog.csdnimg.cn/6f8544ef82704d1d86cf5d2511ade7a8.png)
# 总结
**循环goroutine闭包不能直接使用循环的index、key、value，因为这些变量没有被保存到goroutine栈中，比如以下代码，三个循环内的goroutine都不会得到我们所预期的结果。**
```go
func main() {
	nums := []int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
	for index, value := range nums {
		go func() {
			fmt.Printf("slice index: %d\n", index)
			fmt.Printf("slice value: %d\n", value)
		}()
	}

	m := map[string]string{"1": "1", "2": "2", "3": "3"}
	for key, value := range m {
		go func() {
			fmt.Printf("map key: %s\n", key)
			fmt.Printf("map value: %s\n", value)
		}()
	}

	for i := 0; i < 10; i++ {
		go func() {
			fmt.Printf("range i: %d\n", i)
		}()
	}

	time.Sleep(time.Microsecond * 1000000)
}
```

**我写了个静态扫描工具，针对这种代码写了相应的规则，能扫描出问题，欢迎使用，Github地址：https://github.com/HandsomeDong/xianggolint**