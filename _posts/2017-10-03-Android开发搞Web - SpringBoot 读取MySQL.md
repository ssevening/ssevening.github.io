---
title: Android开发搞Web - SpringBoot 读写MySQL
category: 网站技术
feature_image: "https://raw.githubusercontent.com/ssevening/ssevening.github.io/master/assets/android.png"
image: "https://raw.githubusercontent.com/ssevening/ssevening.github.io/master/assets/android.png"
---

欲穷千里目，更上一层楼。这篇文章讲 SpringBoot 读写MySQL数据库。


<!-- more -->

#### 一、背景
[上一篇文章](http://www.jianshu.com/p/5e4f89a960e0)，介绍了怎样搭建一个web微服务，通过硬编码的方式，可以实现自建API服务，但静态API，逼格肯定不够高，欲穷千里目，更上一层楼。这篇文章讲 SpringBoot 读写MySQL数据库。

#### 二、环境准备
JAVA、Gradle、git、MYSQL。
MYSQL环境稍麻烦一点，还要在本地装MYSQL，简单一点，索性直接去买一个吧。比如：[西部数码](http://www.west263.hk/services/webhosting/database.asp?tabindex=mysql)，80块钱，买一年，想学习嘛，就要舍得下本。购买后，可以得到以下几个内容：地址、数据库名、用户名、密码。

![MYSQL参数](http://upload-images.jianshu.io/upload_images/5649240-d7c5bfe961a4eef8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

舍不得花钱，也可以在自己的机器上装mysql。[MAC安装MYSQL](http://www.jb51.net/article/103841.htm)。

#### 三、开始用Spring boot 读取MYSQL，[参考文档](https://spring.io/guides/gs/accessing-data-mysql/)

老样子，执行下方命令：```git clone https://github.com/spring-guides/gs-accessing-data-mysql.git```  截图如下：

![下载源码截图](http://upload-images.jianshu.io/upload_images/5649240-08edfd0470dafb36.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后用IDEA 导入 工程截图如下：

![导入截图](http://upload-images.jianshu.io/upload_images/5649240-0e612b8425a8bc51.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

继续一路OK下去，等待同步。

示例工程中，```build.gradle```文件中，增加了对MYSQL操作的依赖


![MYSQL依赖](http://upload-images.jianshu.io/upload_images/5649240-7a1ba0c689e202eb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后我们修改 ```application.properties``` 文件，修改为刚购买的MYSQL数据库网址，数据库和用户名。

![修改数据库参数](http://upload-images.jianshu.io/upload_images/5649240-5eeecbb122608831.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后启动服务！！！是的，没错，我们又可以直接开车了。执行命令：```./gradlew bootRun``` 截图如下

![启动 bootRun](http://upload-images.jianshu.io/upload_images/5649240-ac0b9b9ab467e4e1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后访问：```http://localhost:8080/demo/add?name=ssevening&email=ssevening@gmail.com``` 向数据库中添加一条记录


![添加记录成功](http://upload-images.jianshu.io/upload_images/5649240-65fc5ed63ef3d02e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

再访问： ```http://localhost:8080/demo/all``` 得到如下提示：

![网页查看添加的数据](http://upload-images.jianshu.io/upload_images/5649240-8c7a169981ff3149.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

发现也成功了。

再登陆到数据库中去查看：

![数据库中已经存进去了](http://upload-images.jianshu.io/upload_images/5649240-f4f1ae6a1383002c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

不知道你没有注意到以下两个问题：
1. 数据库表结构是什么创建的？他怎么知道要创建什么表？
2. 我是怎么知道要访问 add 和 all 网址的？

#### 四、开始解答

1. 数据库表结构是什么创建的？他怎么知道要创建什么表？

Android开发中，常会用到GreenDao，然后根据实体体去生成创建表的语句，这里也是通过读取注册，直接操作DB执行建表操作。实体类截图如下：
![声明实体类](http://upload-images.jianshu.io/upload_images/5649240-658e4e7b8f51ab53.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后 上面 ```application.properties``` 中的 ```spring.jpa.hibernate.ddl-auto=create``` 即代表在运行时，根据POJO类自动创建数据库表结构。

对于客户端而言，省去了什么？省去了你去写建表语句，做得绝一点，你直接声明好POJO类，启动一次创建好相应的表结构，再把 create 的值改为none，后面就专心写你的业务就可以了。

2. 我是怎么知道要访问 add 和 all 网址的？

启动时，Spring boot 启动日志告诉我们，映射 /demo/add 到 hello.MainController.addNewUser(java.lang.String,java.lang.String)，如下截图：

![映射截图](http://upload-images.jianshu.io/upload_images/5649240-801112a130eec1ce.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

所以我们看一下：```MainController``` 的代码如下：通过代码，我们发现 GET请求，带上参数 name 和 email 会调用 ```userRepository.save();``` 保存用户信息到DB。而 all 则 调用 ```userRepository.findAll()```方法返回相应的JSON数据。


![屏幕快照 2017-10-04 00.26.30.png](http://upload-images.jianshu.io/upload_images/5649240-f9d07917da39fdfc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

而 ```userRepository``` 则通过 Spring 的 ```Autowired``` 织入到Controller中。一切就是这么简单。

至此，基于Spring Boot 读写MySQL数据库就搞定了，自己写几个Bean，去玩一玩吧！

最后一点，花点钱，不寒碜！不一定非走西部数码购买，只是我以前帮朋友修改网站时，恰巧有需求，就买了一个。

#### 五、总结

* 文章看到这里，你已经会用Spring Boot 读写 DB数据了，也就是可以提供动态的API数据了，但现在的数据只能返回JSON或String。仰望天空，还是有一种淡淡的忧伤！如果我想做个页面，或者有图片要显示，怎么办？只会这些，还是没有那么自信！

* 下一篇，介绍：Spring Boot 制作WEB UI界面。

欢迎关注作者微信公众号，及时获得作者更新：

![微信公众号](https://ssevening.github.io/assets/weichat_qrcode.jpg)

另外还建立了小密圈：圈主 和 嘉宾 都是业内顶尖开发者，开发的app被Google 编辑推荐，对性能，架构，图片，MD设计都有研究和深入，欢迎大家加入，提升自己，一起进步，互相帮助交流！

![小密圈](https://ssevening.github.io/assets/mi_qrcode.png)