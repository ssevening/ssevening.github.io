---
title: Android开发搞Web-SpringBoot 添加定时器
category: 网站技术
feature_image: "https://raw.githubusercontent.com/ssevening/ssevening.github.io/master/assets/android.png"
image: "https://raw.githubusercontent.com/ssevening/ssevening.github.io/master/assets/android.png"
---

昨天我们说了，发布了一个应用，老牛逼了，然后用户量扛扛地涨，风投追着屁股送钱，然后继续做梦，但因为用户量上涨，我们带来的问题是：一台服务器，当我们要升级新功能发布新接口的时候，线上用户就直接拒绝服务了，这样事情，肯定不能忍啊，那怎么办呢？我们可有以下几个思路。

<!-- more -->

#### 背景

昨天我们说了，发布了一个应用，老牛逼了，然后用户量扛扛地涨，风投追着屁股送钱，然后继续做梦，但因为用户量上涨，我们带来的问题是：一台服务器，当我们要升级新功能发布新接口的时候，线上用户就直接拒绝服务了，这样事情，肯定不能忍啊，那怎么办呢？我们可有以下几个思路。

#### 天马行空

1. 切域名解析，因为都是通过域名来访问API的，我们买两台独立IP的机器，发新版本时，先切DNS解析到新的IP，然后升级程序，再把DNS切换回来。 【这样的馊主意估计没有人会采纳吧！DNS解析要24小时生效啊！果断放弃】

2. 客户端内置两个API域名，一个调用失败时，自动切换到另外一个，就是传说中的备胎，一个女人，找一个男朋友，然后再留个备胎，男朋友不在的时候，备胎补上。

3. 客户端配置一个超级稳定无敌的获取接口IP的接口，然后每次启动，先获取最新的网关IP，再发送网络请求，这样在换机器的时候，超级无知识稳定版里面去切换IP。

4. 皮条客模式，需要特殊服务时，先找皮条客，然后皮条客知道每一个服务人员的健康状态，在线状态，例假状态，然后决定把这个请求分配给谁。这种模式，嫖客省心了，只要找皮条客就可以了，皮条客负责分配自己的需求下去，一个皮条客后面是一堆鸡群，所以暂时不用担心皮条客被压跨了。

#### SpringBoot Ribbon 皮条客模式开启

1. 首先准备环境：gradle、git、java、docker、IDEA

2. 下载代码，开搞 ```git clone https://github.com/ssevening/gs-client-side-load-balancing.git```
老样式，不行就自己fork一份再copy.


![下载代码](http://upload-images.jianshu.io/upload_images/5649240-7a150ba0fead3864.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

下载完代码后，我们可以看到，一个 ```user``` 文件夹，一个 ```sayhello``` 文件夹。
这里面是两个应用。可以理解为：USER会员服务中，有一项服务是 ```sayhello```，然后USER会员服务组一个集群，建立多个 sayhello服务，注册到USER集群中。

所以，我们首先，启动User服务。如下载图：


![先启动集群](http://upload-images.jianshu.io/upload_images/5649240-1bbdd317970b5e08.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后我们再去启动 sayhello服务。如下截图：

![启动Client](http://upload-images.jianshu.io/upload_images/5649240-79c69053abd191c1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这样我们启动了一个集群，同时挂了一台服务，那我们再来挂两台服务，
9999端口开一个服务
```SERVER_PORT=9999 ./gradlew bootRun```
9092也开一个sayhello服务。
```SERVER_PORT=9092 ./gradlew bootRun```

这样，我们一共开启了四个服务，一个User集群，然后三个 sayHello服务。分别的启动端口是:
```localhosts:8080, localhosts:9092,localhost:9999```



![启动四个服务](http://upload-images.jianshu.io/upload_images/5649240-bb89558172dda339.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后我们来验证一下，我们的集群服务吧。

首先访问：```http://localhost:8888/ssevening?name=PanChenXing```，然后多刷新几次，大概的截图如下：

![第一种样子](http://upload-images.jianshu.io/upload_images/5649240-1686fe4be9574d54.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![第二种](http://upload-images.jianshu.io/upload_images/5649240-d23ee72f3e4946cd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![第三种](http://upload-images.jianshu.io/upload_images/5649240-8c3a86221b9931d9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


然后多刷新几次，去后台看日志，先看User模块的日志如下：
![User集群日志](http://upload-images.jianshu.io/upload_images/5649240-97c6725c3e3683df.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

注意看：```Instance count:3``` 有三个实例，以及三个实例对应的端口 8090,9092 9999 。可以理解为一个集群的管理。

再看第一台sayHello的日志


![第一台日志](http://upload-images.jianshu.io/upload_images/5649240-acd6cb3bc8257770.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

已经在受理 ssevening 相关请求的服务了，即：鸡开始接客了。

另外两台也是类似的日志。

这个时候，我们停掉第二和第三台。再去看集群的日志，首先会有异常抛出来如下：

![检查集群服务，结果超时了](http://upload-images.jianshu.io/upload_images/5649240-3da8f65ce89f453a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这个时候，再去访问：[http://localhost:8888/ssevening?name=PanChenXing](http://localhost:8888/ssevening?name=PanChenXing)，你会发现服务还正常运行。

然后另外两台改一点点代码，再发布上去：比如，强制都返回:```Android can do web work```


![改代码，发版本](http://upload-images.jianshu.io/upload_images/5649240-75e8d65e13c226f1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


通过```SERVER_PORT=9092 ./gradlew bootRun``` 启动起来后，再去停掉：第一个sayhello服务。


![可以看到服务提供方已经被替换了](http://upload-images.jianshu.io/upload_images/5649240-51b03c26a9a43bc9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这样子，我们再发布服务的话，就可以在保证业务不间断的前提下，来升级API了。

#### 揭开集群代码的神秘面纱。

首先，我们看User集群的代码。依赖就不看了，自己下载源码吧。

我们讲重要的部分：集群中有哪些机器在哪里配的？

![集群配置](http://upload-images.jianshu.io/upload_images/5649240-6623d8ca6f00a375.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

通过上面的代码，代表我们会启三个sayhello服务，然后通过user来管理。那具体的功能实现呢？见下面代码：


![服务实现，皮条件干的活](http://upload-images.jianshu.io/upload_images/5649240-1a6d8f96019611bf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这里所有的调用都是通过restTemplate 去实例列表中获取一个服务后执行的调用。完成了所谓的集群。

看完了集群端，我们再看client端。


![最简单的web服务，没有什么特别](http://upload-images.jianshu.io/upload_images/5649240-b3e094069ec5143c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

所以综合下来说，就是：对过皮条客：ribbon 完成了一次服务分配和提供的功能。

这个时候，喜欢上云的人，肯定跃跃欲试了，我要发布到云上去，然后到处去部署。

传docker的事，我这里就不做了。

在user和 sayhello中分别运行： ```./gradlew buildDocker``` 构建到本地docker中。

![Docker中的镜像文件](http://upload-images.jianshu.io/upload_images/5649240-0f3a67e368bb4bd4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

开始运行实例：

先把docker 中的 User集群启动起来。

```docker run -p 80:8888 -t  ssevening/gs-spring-boot-load-banlanceing-user /bin/bash```


![启动docker中的User](http://upload-images.jianshu.io/upload_images/5649240-0174548527ba8fcd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

再搞起一台客户端来：```docker run -p 8090:8090 -t ssevening/gs-spring-boot-load-banlanceing-say-hello /bin/bash```


![启动sayHello服务](http://upload-images.jianshu.io/upload_images/5649240-d8c72d8f4caeaf19.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这样子尝试了一把，但最终失败了，因为在docker中配置的localhost:8090，没有办法跳出docker容器来再访问我们外面的localhost，所以要折腾的话，建议改成阿里云的VIP，如：123.456.789.1:8090 ，就可以使用集群服务了。




#### 一些总结

想要24小时不间断服务的目标可以说是基本完成了，为什么是基本呢？因为还是太人肉了，要自己配要开哪些端口，要自己配网络IP，万一集群要增加机器，那还是要重启整个集群，这个有点恶吧？那怎么样把集群搞稳定？不因为增加或删除机器而重启呢？

那就要用到发现服务，让客户端把所有的服务都注册到一个地方：eureka ，然后User集群自动去eureka读取最新的服务列表，这样会不会更爽呢？相当于皮条客在红灯区放了一个 打卡机，每天定时打卡，就知道有谁没有来，有谁可以提供服务了。

下一次，搞 eureka .

参考资料：[https://spring.io/guides/gs/client-side-load-balancing/](https://spring.io/guides/gs/client-side-load-balancing/)

欢迎关注作者微信公众号，及时获得作者更新：

![微信公众号](https://ssevening.github.io/assets/weichat_qrcode.jpg)

另外还建立了小密圈：圈主 和 嘉宾 都是业内顶尖开发者，开发的app被Google 编辑推荐，对性能，架构，图片，MD设计都有研究和深入，欢迎大家加入，提升自己，一起进步，互相帮助交流！

![小密圈](https://ssevening.github.io/assets/mi_qrcode.png)