---
title: Android开发搞Web - 阿里云ECS Docker搭建WEB
category: 网站技术
feature_image: "https://raw.githubusercontent.com/ssevening/ssevening.github.io/master/assets/android.png"
image: "https://raw.githubusercontent.com/ssevening/ssevening.github.io/master/assets/android.png"
---

今天我们就来完成在阿里云ECS服务器上，安装Docker，提交Docker 镜像到 docker hub，以及阿里云ECS下载镜像并运行的例子。

<!-- more -->

#### 背景
上一篇文章，我们确定了[WEB容器为Docker](http://www.jianshu.com/p/bafefa6de837)，今天我们就来完成在阿里云ECS服务器上，安装Docker，提交Docker 镜像到 docker hub，以及阿里云ECS下载镜像并运行的例子。

#### 环境准备
阿里云帐号，支付宝的钱。

#### 搞阿里云ECS服务器。

进入阿里云ECS后台截图如下：
![进入阿里云ECS后台](http://upload-images.jianshu.io/upload_images/5649240-bc2ab99bb95e1cdc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

点击创建实例按钮后截图如下：

![选择主机位置](http://upload-images.jianshu.io/upload_images/5649240-ad664b6f315f52e7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![配置网络和安全组，勾选80端口](http://upload-images.jianshu.io/upload_images/5649240-71c9bc903279f12a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![操作系统这里选ubuntu](http://upload-images.jianshu.io/upload_images/5649240-b1db4401dbd2c75f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![因为是测试使用的，时间就选一周吧](http://upload-images.jianshu.io/upload_images/5649240-6c5918c5bd200d9f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

最后一定要记好你设置的密码噢！

订单确认页面截图如下：

![订单确认页面](http://upload-images.jianshu.io/upload_images/5649240-44d4e28971b83759.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

其他就付款，然后等着阿里云创建实例就可以了。

然后再到实例列表中，截图如下，机器正在启动中了。


![等一会吧，机器启动好了，我们就可以登上去了](http://upload-images.jianshu.io/upload_images/5649240-2f53cf8b70c958ad.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

注意看截图中的公网IP，可以从mac上ssh上去。

执行命令: ```ssh root@116.62.240.150``` 再输入你刚才设置的密码，登录到机器上。截图如下：

![登录到机器上](http://upload-images.jianshu.io/upload_images/5649240-c34ebd450b6ff396.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

到这里，阿里云的ECS就算搞好了。记好用户和密码噢！

#### ECS上安装Docker

因为是[Ubuntu系统](https://www.docker.com/docker-ubuntu)，我们先更新一下源。

```
sudo apt-get update
sudo apt-get install docker
sudo apt-get install docker.io
docker -v
```
看到如下截图代表安装成功：

![查看docker版本](http://upload-images.jianshu.io/upload_images/5649240-d0059d588145cbd4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这么简单，我们就又把Docker安装完成了，下面就是给Docker跑应用了。

####  Docker上跑个简单的应用

执行命令：```docker pull springio/gs-spring-boot-docker``` spring boot 开发好的一个lib镜像。截图如下：

![下载成功，有可能比较慢](http://upload-images.jianshu.io/upload_images/5649240-ea2c7b6acfc92675.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


然后我们就可以执行命令：```docker run -p 80:8080 -t  springio/gs-spring-boot-docker /bin/bash``` 用主机的80端口映映docker中的8080端口，这样，就可以直接访问公网IP对应的网址了。截图如下：


![下载和启动WEB](http://upload-images.jianshu.io/upload_images/5649240-93609fc87981b9f1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


现在来访问一下我们的spring-boot官方未例吧！[http://116.62.240.150/](http://116.62.240.150/)，访问截图如下：

![访问成功](http://upload-images.jianshu.io/upload_images/5649240-80d1f946dbbdb817.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


讲到这里，是不是很惊讶？靠，这么简单？和我们拉代码一样简单，我和我的小伙伴都震惊了呢！但我还不满足啊！刚才跑的是springio的例子，我要怎样才能传到仓库一个自己的镜像呢？不要急，继续向下看！

#### 上传镜像到Docker

还记得上个版本中，你拉下来的代码吗？在这里再拉一次吧。

执行命令：```git clone https://github.com/ssevening/SpringBootWithDocker.git```

如果失败，就fock到自己的github上。

然后在本机上安装好docker。详见：[MAC系统装Docker](http://www.jianshu.com/p/97268959ac64)

1. 没帐号，去注册帐号，然后通过命令行 ```docker login``` 登陆。

![看提示，没帐号去注册帐号.png](http://upload-images.jianshu.io/upload_images/5649240-08466999594eeda1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2. 到SpringBootWithDocker目录，执行 ```./gradlew build buildDocker``` 发布镜像到本地Docker中。

![发布镜像到本机Docker中](http://upload-images.jianshu.io/upload_images/5649240-c5a8dce593346801.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

运行脚本：```docker images```，下如下截图：

![最上面一个代表本地刚刚添加的版本](http://upload-images.jianshu.io/upload_images/5649240-a0066cafaa652cdb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
运行脚本```docker tag 20b0e289af0b ssevening/gs-spring-boot-docker:v1.0``` 重命名一个v1.0的tag.

![重命名tag](http://upload-images.jianshu.io/upload_images/5649240-3f0f80c402bb5c92.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

运行脚本：```  docker push ssevening/gs-spring-boot-docker:v1.0``` 截图如下：



![推送到远程镜像](http://upload-images.jianshu.io/upload_images/5649240-3af9cbd3b0dad6d8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![远程镜像图片，有1.0的了](http://upload-images.jianshu.io/upload_images/5649240-c28281867b95ec29.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


成功后，就可以到阿里云机器上去 pull 下镜像来了噢！我们远程登到阿里云的机器上，然后下载 执行 ```docker pull ssevening/gs-spring-boot-docker``` 下载镜像，再运行（需要有localhost的MySQL数据库）


![阿里云服务器上下载镜像](http://upload-images.jianshu.io/upload_images/5649240-5e16dc44d8dc18d0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


参考资料：[Spring boot with docker](https://spring.io/guides/gs/spring-boot-docker/)
[删除镜像](http://www.simapple.com/341.html)

欢迎关注作者微信公众号，及时获得作者更新：

![微信公众号](https://ssevening.github.io/assets/weichat_qrcode.jpg)

另外还建立了小密圈：圈主 和 嘉宾 都是业内顶尖开发者，开发的app被Google 编辑推荐，对性能，架构，图片，MD设计都有研究和深入，欢迎大家加入，提升自己，一起进步，互相帮助交流！

![小密圈](https://ssevening.github.io/assets/mi_qrcode.png)