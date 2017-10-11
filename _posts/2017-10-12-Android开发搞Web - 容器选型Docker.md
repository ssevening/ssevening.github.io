---
title: Android开发搞Web - 容器选择Docker
category: 网站技术
feature_image: "https://raw.githubusercontent.com/ssevening/ssevening.github.io/master/assets/android.png"
image: "https://raw.githubusercontent.com/ssevening/ssevening.github.io/master/assets/android.png"
---

我们已经可以做出简单的API，简单的页面，然后内容都有了，我们要给我们的WEB找一下安身之所，生了姑娘总要嫁出去吧！总不能放在localhost里面，然后当个老姑娘吧！

<!-- more -->

#### 背景
继上一篇，[Android开发搞Web-Spring Boot 制作Web UI界面](http://www.jianshu.com/p/500550b0d502)，我们已经可以做出简单的API，简单的页面，然后内容都有了，我们要给我们的WEB找一下安身之所，生了姑娘总要嫁出去吧！总不能放在localhost里面，然后当个老姑娘吧！

#### 部署标准

1. 【环境稳定】  我们选用：阿里云 的ECS服务器
2. 【业内标准】 应用上云后，后期不管迁移到AWS，还是腾讯云，都一样了。
3. 【易扩展】 上云后，每一台机器，都涉及到Tomcat, MySQL,JAVA,Ubuntu 等一系列环境，实在是太麻烦，这个问题要解决。
4. 【易部署】 上面解决了环境问题，那我们的代码怎么上传到云上呢？直接通过scp 传递一个 war 包到云服务器，然后再启动应用，如果发布的war包有问题，要回滚，还要去自己再去找历史的war包文件回滚回来。这个问题也要解决。

#### Docker神器 + 阿里云 ECS 一统江湖


![Docker配图，容器装载一切](http://upload-images.jianshu.io/upload_images/5649240-750336764262ac49.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

1. 什么是Docker?
第一次听说这个名词，我也蒙逼了，如果你也是初次接触，那就听我来打个比喻吧！

近期房地产比较火，那我们就拿房地产来做个例子，首先Docker是一个容器，里面可以装很多东西。

开发商首先要从政府那里买地过来(有了一台服务器硬件设备)，然后开发商要在上面盖小区（开发商就相当于Docker），开发商固定售卖 A B C D 四种户型(Docker中的Image镜像)，最终盖好的房子(相当于镜象的实例)，这些实例共享小区中的公共配套比如：游泳池，篮球馆 (服务器上共享的内存、硬盘、网络)。

盗梦空间相信大家都看过吧? 四层嵌套的梦，在Docker中，通常只是在服务器上进入第一层梦境，然后发生了一系列事情。

那有了上面的Docker以后，有什么好处呢？

1.  我们先从购买房子的用户角度出发：到售楼处后，说给我来一套A户型的房子，交钱后，就等着开发商帮你建房子就行了。换到程序员视角就是：我有一个机器，我装上一个docker，告诉Docker，给我来一个JAVA，给我再来个MySQL，然后只需要键入命令行:```docker pull mysql```  MySQL就安装好了。就是这么省心。这样第三个问题解决了，环境易配置。

2. 环境易配置后，那我们再来解决易部署的问题，用过Git的都知道，我们提交的代码，都是有版本控制的，那我们的WAR包，有没有可能也用版本管理来做呢？答案是可以的，详见：[Docker Hub](https://hub.docker.com/)，我们可以通过命令行 ```docker build -t="gs-spring-boot-docker-0.1.1" .``` 构建到本地Docker镜像中，然后再通过 ```docker push ssevening/gs-spring-boot-docker:0.1.1``` 提交到远程仓库。这个时候，任意安装有docker的服务器，只要运行 ```docker pull ssevening/gs-spring-boot-docker:0.1.1``` 就可以把代码给下载下来，然后再通过 ```docker run -p 80:8080 -t  ssevening/gs-spring-boot-docker /bin/bash``` 即可启动我们的应用了噢！

所以，果断选用Docker走起！

那快点去安装一下呗。[Mac 安装Docker](http://www.jianshu.com/p/97268959ac64)

安装好了，我们就开始在阿里云上跑WEB应用，以及MySQL，绑定域名等一系列的事情了。
下一个战场：阿里云服务器购买和安装Docker。


![阿里云](http://upload-images.jianshu.io/upload_images/5649240-ee48f9fff9c518aa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

欢迎关注作者微信公众号，及时获得作者更新：

![微信公众号](https://ssevening.github.io/assets/weichat_qrcode.jpg)

另外还建立了小密圈：圈主 和 嘉宾 都是业内顶尖开发者，开发的app被Google 编辑推荐，对性能，架构，图片，MD设计都有研究和深入，欢迎大家加入，提升自己，一起进步，互相帮助交流！

![小密圈](https://ssevening.github.io/assets/mi_qrcode.png)