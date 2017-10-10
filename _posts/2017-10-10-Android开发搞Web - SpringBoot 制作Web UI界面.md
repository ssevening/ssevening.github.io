---
title: Android开发搞Web - SpringBoot 制作Web UI界面
category: 网站技术
feature_image: "https://raw.githubusercontent.com/ssevening/ssevening.github.io/master/assets/android.png"
image: "https://raw.githubusercontent.com/ssevening/ssevening.github.io/master/assets/android.png"
---

接上一篇文章：SpringBoot读写MySQL 后，我们已经可以从DB中动态的输出 数据库 中的数据给客户端使用。

<!-- more -->

#### 背景
接上一篇文章：[SpringBoot读写MySQL](http://www.jianshu.com/p/3ddd0e74f6f0) 后，我们已经可以从DB中动态的输出 数据库 中的数据给客户端使用。但这些还不够，因为客户端用到的不仅仅是API，还有WEB H5的页面。所以今天我们就来搭一个Spring Web页面。

#### 环境准备
还是老三样，如果你前面跟着走过来的，那直接可以跳过了。
git，IDEA，Gradle

#### 开始下载和运行WEB代码

援人以鱼不如授人以渔，那我们直接把Spring-boot的所有示例代码放出来：[SpringBoot](https://github.com/ssevening/spring-boot/),截图如下：

![代码示例](http://upload-images.jianshu.io/upload_images/5649240-d1c433f6211c814d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

上方截图中，看到有 [spring-boot-samples](https://github.com/ssevening/spring-boot/tree/master/spring-boot-samples/)，直接点进去，可以看到下面一堆堆的示例！


![想学啥有啥，简单到不像话](http://upload-images.jianshu.io/upload_images/5649240-5407419357391c2c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这里我们写WEB页面，那就搜索：[web-ui](https://github.com/ssevening/spring-boot/tree/master/spring-boot-samples/spring-boot-sample-web-ui)

下载我已写好的示例工程,然后通过IDEA导入工程。
```
git clone https://github.com/ssevening/SpringBootWithDocker.git

```

![确保数据库相关内容配置正确](http://upload-images.jianshu.io/upload_images/5649240-cbde9e2313eb19a0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如果有数据库，就修改为自己的数据库网址和数据库名称。


如果没有，连接上去，然后创建相应的DB

```
连接mysql: docker exec -it mysql bash

创建数据库：create datebase ssevening;
```

然后就可以到目录下去启动应用了，截图如下：

![直接启动应用截图](http://upload-images.jianshu.io/upload_images/5649240-f6bcb96fe34d2db3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

至此，我们就可以测试一下我们写的页面了。首先访问: [http://localhost:8080/Index.html](http://localhost:8080/Index.html), 访问后得到如下截图：

![首页访问截图](http://upload-images.jianshu.io/upload_images/5649240-b655894a8722787a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![点击here后的页面](http://upload-images.jianshu.io/upload_images/5649240-fb5b51e862b72640.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![调用API添加用户](http://upload-images.jianshu.io/upload_images/5649240-7d05afc6ae6c5868.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![通过页面访问刚刚添加的用户](http://upload-images.jianshu.io/upload_images/5649240-568a23565ff533cd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


通过上面的几个页面，代表我们的：API服务，DB服务，静态资源网页，动态网页，都搞好了。
下面我们来看一下相应的代码。

#### WEB代码分析：

首先，我们看一下Gradle的依赖：


![增加了WEB的依赖](http://upload-images.jianshu.io/upload_images/5649240-4ad4d7ad383448a5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

添加好相应的application配置文件，SpringBoot讲究的是按约定编程，按它的规矩办事，一切就都运行好好的，把相应的文件都放正确的位置，一切也就运行好好的。


![涉及到的文件如下](http://upload-images.jianshu.io/upload_images/5649240-992571409a09d27e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后我们先看欢迎页：[http://localhost:8080/Index.html](http://localhost:8080/Index.html)的代码。


![非常简单，就直接是Html的代码](http://upload-images.jianshu.io/upload_images/5649240-d5fac59dd43d147f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后再看链接到的：[http://localhost:8080/greeting?name=ssevening](http://localhost:8080/greeting?name=ssevening)的代码。


![可以看到处理greeting的方法](http://upload-images.jianshu.io/upload_images/5649240-d9b37b049c09b52e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

需要传递参数为:name,默认值World，这里我们传递的是ssevening.
然后向model中，传递了一个name参数，到页面中。
而```return greeting``` 代表渲染模板是 greeting.html，我们也来看一下代码。


![greeting.html的代码](http://upload-images.jianshu.io/upload_images/5649240-73e4face5074aa68.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这样子，在页面中，直接通过 ```${name}``` 显示出我们参数传递的值。

通过参数传递，在页面上显示，这很正常，JS都可以做到这功能，那我们就换一种，通过DB查询返回当前用户的列表，我们看```greetingall``` 对应的代码如下：


![greetingall 的代码](http://upload-images.jianshu.io/upload_images/5649240-142cd34716a98f03.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这里通过查询数据库，然后把所有的用户都放到了users对象中，然后通过 ```greetingall.html```  来渲染页面。


![greetingall 模板渲染](http://upload-images.jianshu.io/upload_images/5649240-09b6e54423590843.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

上面直接把出对象，然后画了一个表格，循环把数据取出来！完成一次最简单的WEB DB的交互。

#### 总结

看到这里，我们的小目标算是完成了一大步了，现在我们学会了搭API服务，通过API读取DB的数据，通过页面访问DB的数据，满足日常Android的开发需求，不成问题，但哥不喜欢丑逼的 [http://localhost:8080/Index.html](http://localhost:8080/Index.html) ，跑在本地，太Low了，好不容易做出来的东西，怎么也要部署到外网可以给别人访问到吧？

好，那下一篇，我们来讲应用的布署。

欢迎关注作者微信公众号，及时获得作者更新：

![微信公众号](https://ssevening.github.io/assets/weichat_qrcode.jpg)

另外还建立了小密圈：圈主 和 嘉宾 都是业内顶尖开发者，开发的app被Google 编辑推荐，对性能，架构，图片，MD设计都有研究和深入，欢迎大家加入，提升自己，一起进步，互相帮助交流！

![小密圈](https://ssevening.github.io/assets/mi_qrcode.png)