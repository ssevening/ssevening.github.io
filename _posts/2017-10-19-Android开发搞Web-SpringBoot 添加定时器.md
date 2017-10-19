---
title: Android开发搞Web-SpringBoot 添加定时器
category: 网站技术
feature_image: "https://raw.githubusercontent.com/ssevening/ssevening.github.io/master/assets/android.png"
image: "https://raw.githubusercontent.com/ssevening/ssevening.github.io/master/assets/android.png"
---

智能时代，解放双手的时代，接上一篇，我们好不容易在[阿里云上部署了一个Web应用](http://www.jianshu.com/p/3df3655932dd)，网站跑得很正常，但通常我们会产生以下需求：
1. 数据索引建立，比如每一小时更新一次搜索的索引。比如你写的一篇文章，要第二天或第三天才会被百度搜索到，就是说百度每天会更新一次自己的搜索索引。
2. 健康检查，定时检查某些服务是否可用，在不可用时，发短信提醒。
3. 其他需要跑的定时任务。

<!-- more -->

#### 背景

智能时代，解放双手的时代，接上一篇，我们好不容易在[阿里云上部署了一个Web应用](http://www.jianshu.com/p/3df3655932dd)，网站跑得很正常，但通常我们会产生以下需求：
1. 数据索引建立，比如每一小时更新一次搜索的索引。比如你写的一篇文章，要第二天或第三天才会被百度搜索到，就是说百度每天会更新一次自己的搜索索引。
2. 健康检查，定时检查某些服务是否可用，在不可用时，发短信提醒。
3. 其他需要跑的定时任务。

#### 我们的要求

1. 基于上述的说法，我们要在我们的应用中添加一个定时器，定时处理用户上传上来的文件，定时去检查一下百度网站是不是挂了。所以我们有了定时的需求。

#### 直接开撸

启动MYSQL：```docker run --name ssevening-mysql -p 12345:3306 -e MYSQL_ROOT_PASSWORD=123456 -d mysql:latest```

如果没有创建数据库，那就登陆上去创建一把
```docker exec -it ssevening-mysql bash```

登陆MYSQL
```mysql -u root -p```

```create database ssevening;```

然后下载已写好的仓库代码：

```git clone https://github.com/ssevening/SpringBootWithDocker.git```

运行代码截图如下：


![spring boot 定时器](http://upload-images.jianshu.io/upload_images/5649240-ffd12075d3f6830b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后通过命令行，我们就可以看到我们已经在后台执行的定时任务了。


![定时任务](http://upload-images.jianshu.io/upload_images/5649240-2c58ca9efefe53ef.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后我们再看代码，配一个定时任务需要哪些代码？

1. 不需要增加任何依赖。
2. 修改Application代码，增加```@EnableScheduling // 开启定时执行任务功能```

![增加定时任务功能](http://upload-images.jianshu.io/upload_images/5649240-c1d4da9544e1b7fc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

3. 定时器代码截图如下：可以通过：[cron格式生成器](http://cron.qqe2.com/) 来生成你想要的时间。比如通过短信或邮件的方式，每半小时提醒自己起来走动走动，虽然有点重，一个小小的Android闹钟就可以解决的问题。


![定时器代码](http://upload-images.jianshu.io/upload_images/5649240-4e4f63e2d40bc119.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#### 撸好了代码，发布到仓库

编译镜像
```./gradlew build buildDocker```


![编译镜像](http://upload-images.jianshu.io/upload_images/5649240-8d68314009856868.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

重命名和推送镜像到远程：
```docker tag 8eb2e8586d9a ssevening/gs-spring-boot-docker:v2.0.timer```

```docker push ssevening/gs-spring-boot-docker:v2.0.timer```

![推送到远程](http://upload-images.jianshu.io/upload_images/5649240-73d0343d9c99b129.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

现在我的仓库中就有一个 timer的版本了。


![上库成功](http://upload-images.jianshu.io/upload_images/5649240-8055f8aeeefd34ce.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

后期就可以通过：```docker rmi -f 8eb2e8586d9a``` 删除我们本地的镜像，然后从远程拉下我们最新的镜像运行一把。


![Docker镜像下载](http://upload-images.jianshu.io/upload_images/5649240-f829e5d234459e52.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![慢慢等下载成功吧](http://upload-images.jianshu.io/upload_images/5649240-dd0075500d824d82.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


下载成功，运行行一把
```docker run -p 80:8080 -t  ssevening/gs-spring-boot-docker /bin/bash```

然后我们的应用就牛逼了，然后用户量就来了，然后要求24小时不间断服务了，就算新版本发布也不能停机器了，这可怎么办？下一篇文章想一想，做一做，试一试吧！