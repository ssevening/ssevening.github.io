---
title: Android开发学WEB-MAC 安装 Docker
category: 网站技术
feature_image: "https://raw.githubusercontent.com/ssevening/ssevening.github.io/master/assets/android.png"
image: "https://raw.githubusercontent.com/ssevening/ssevening.github.io/master/assets/android.png"
---

2008年工作时，流行的J2EE还是SSH，工作后，Ibatis和 MyIbatis 替代了Hibernate，因为自己写SQL，性能更好，然后Spring MVC(Spring一家独大了)，又因为移动互联网的发展，REST 微服务开始流行，Spring又推出了Spring boot。


<!-- more -->

#### 一、背景

2008年工作时，流行的J2EE还是SSH，工作后，Ibatis和 MyIbatis 替代了Hibernate，因为自己写SQL，性能更好，然后Spring MVC(Spring一家独大了)，又因为移动互联网的发展，REST 微服务开始流行，Spring又推出了Spring boot。

前几年移动互联网浪潮比较大，把我也吹到Android端，好几年没有碰服务端的东西，现在猛的回过头来，再看看这几年Server开发，真是日新月异了。

#### 二、先立个小目标，自己搭个WEB服务器。

做Android开发，很多时候，要依赖于服务端接口返回，熟话说：不会做服务端的Android开发，不是好开发！那我们开搞。

可以返回JSON格式方法很多，比如ASP，PHP，JSP，这些都是页面级别的返回，我们不讲。而WEB容器而言，又有IIS, apache,Tomcat，jBoss 等一堆容器。

现在的大型电商都走JAVA，我们这里选用Spring boot, 为什么？因为简单。Spring为了让开发专注于写业务，把框架做到了简单的极致！说到简单，那复杂要复杂到什么程度？以前配置SSH框架，要录个视频来教学。而现在，一个文档就可以搞定了。

不扯了，操作方法来自：[参考文档](https://spring.io/guides/gs/spring-boot/)

先说一下环境：JAVA、Gradle、Git。

这个时候，做为Android开发工程师的你，是不是很开心？这环境哥全都有！有环境就好办，我们接着搞。

运行下方命令行下载官方示例工程：

```git clone https://github.com/spring-guides/gs-spring-boot.git```

![下载源代码截图](http://upload-images.jianshu.io/upload_images/5649240-584828ee119f854f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

下载完成后，执行下方命令启动服务！！！什么？老司机啊，你这车也开得太快了吧？刚下载完代码，就直接运行了？是的，做一个男人，要又快又持久！

```
PandeMacBook-Pro-2:gs-spring-boot Pan$ pwd
/Users/Pan/work/gs-spring-boot
PandeMacBook-Pro-2:gs-spring-boot Pan$ ls
CONTRIBUTING.adoc	LICENSE.code.txt	LICENSE.writing.txt	README.adoc		complete		initial			test
PandeMacBook-Pro-2:gs-spring-boot Pan$ cd complete/
PandeMacBook-Pro-2:complete Pan$ ls
build.gradle	gradle		gradlew		gradlew.bat	mvnw		mvnw.cmd	pom.xml		src
PandeMacBook-Pro-2:complete Pan$ ./gradlew build && java -jar build/libs/gs-spring-boot-0.1.0.jar
Downloading https://services.gradle.org/distributions/gradle-2.13-bin.zip

```

运行后的截图如下：

![屏幕快照 2017-10-03 10.05.16.png](http://upload-images.jianshu.io/upload_images/5649240-59e6123f4bf9516c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后我们访问: ```http://localhost:8080``` 截图如下：

![运行截图](http://upload-images.jianshu.io/upload_images/5649240-7ca1cc05327250a4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

到这里亲已经搭建好一个WEB服务了！


#### 三、这样就说自己会WEB，我自己都说服不了我自己！

我们来看代码。看代码，就要选择编辑器，Android开发都用Android Studio，那WEB开发，我们用IDEA。熟悉的界面，熟悉的味道！

导入工程，然后一路OK下去就可以了。

![导入工程](http://upload-images.jianshu.io/upload_images/5649240-a99a227e7e45c468.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

快来看看我们熟悉的 main方法，这里是程序的入口:

![web程序入口](http://upload-images.jianshu.io/upload_images/5649240-ac2197768b5edb56.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后看都需要哪些依赖呢？

![依赖](http://upload-images.jianshu.io/upload_images/5649240-39e517e09220890b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

看完了上面的东东，是不是想说：我靠，又是熟悉的味道啊！Google 就是这么的屌！

最后，我们来看刚才那一句：Greetings from Spring Boot! 是从哪里来的？我们又可以输出怎样的数据呢？比如输出一个Map，里面放上业务数据，比如当前写代码的程序猿的基本信息。

编写代码如下截图：

![修改部分截图](http://upload-images.jianshu.io/upload_images/5649240-8f2f4c2fbff62b63.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


然后访问: ```http://localhost:8080/whoareyou```，截图如下：


![运行截图](http://upload-images.jianshu.io/upload_images/5649240-e878bb8cbee4553e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


会了上面这些后，已经可以通过代码直接搞一个静态的API数据返回接口，但静态页面，没有DB操作，还是不够欣喜！还是不够兴奋。下一次，我们就讲Spring Boot 操作 mysql 操作。

更多详细的注解功能，去阅读其他人写的文档吧。比如：[springBoot注解大全](http://www.cnblogs.com/tanwei81/p/6814022.html)

欢迎关注作者微信公众号，及时获得作者更新：

![微信公众号](https://ssevening.github.io/assets/weichat_qrcode.jpg)

另外还建立了小密圈：圈主 和 嘉宾 都是业内顶尖开发者，开发的app被Google 编辑推荐，对性能，架构，图片，MD设计都有研究和深入，欢迎大家加入，提升自己，一起进步，互相帮助交流！

![小密圈](https://ssevening.github.io/assets/mi_qrcode.png)