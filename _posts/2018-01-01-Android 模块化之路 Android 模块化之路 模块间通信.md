---
title: Android 模块化之路 模块间通信
category: Android开发
feature_image: "https://raw.githubusercontent.com/ssevening/ssevening.github.io/master/assets/android.png"
image: "https://raw.githubusercontent.com/ssevening/ssevening.github.io/master/assets/android.png"
---

Android 模块化之路 模块间通信

<!-- more -->

#### 一、背景

Android 开发，从最初的一个人团队，我的地盘我做主，随着团队和业务逐渐变大，单App开发慢慢跟不上业务发展步伐。

* 代码复用性： 再牛X的代码，不能给其他团队使用，其他团队无法使用，也不牛X。
* 业务稳定性：代码改动不可控，测试回归不可控，业务不稳定。
* 快速发射小卫星：业务要发布新App，一切从头开始，没有现成组件或模块可共用，刘欢唱起：大不了从头再来？

所以就走上Android 模块化之路。

#### 二、小App向模块化App演变过程

##### 模块化架构改造：

![初期架构到后期架构](http://upload-images.jianshu.io/upload_images/5649240-0b729112c144715b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

1. 把原App中的网络、图片、UIKIT都放到MAVEN仓库。
2. 抽离BaseActivity 和 公共资源到Common中
3. 逐渐下沉业务Module，做到业务隔离。

做到第3步时，就面临着模块间跳转和路由的选型了。所以路由的选型， [传送门 : Android路由选型](http://www.jianshu.com/p/3ff0d55ccdfc)

搞定路由后，可以达到如下图的基础架构

![模块化通用架构](http://upload-images.jianshu.io/upload_images/5649240-4f97cca946de5873.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后新的问题也跟着来了！模块间通信和模块调用。

在前期抽业务模块过程中，如产品模块用到：获取购物车数量，或添加到购物车，就把这两个功能也下沉到Common中，慢慢就有形成一个万能的Common。

所以面临着模块间的通信和调用的选型，问题如下：三个飘着的气球，要把这几个气球给落地。

![要落地的气球](http://upload-images.jianshu.io/upload_images/5649240-59cbe29f02eee926.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* Big Common ： 随着模块拆解越来越多，下沉的业务逻辑也会越来越多，大 Common 吾不想要！

* Muti Module Call: 多模块间调用，不让下沉到Common,那就面临多模块间调用，就要寻找模块间调用的方案。

* Be On Cloud: 在云端，上云，App上云的概念，第一步是让仓库代码可以被大家依赖调用到。而不是说：哥，你把我这代码拷走吧！把我的BigCommon拷走！（人家才不拷，你在依赖别人库时，多一个功能你都不想要呢）


#### 三、模块间调用思路与方案

![模块间调用思路](http://upload-images.jianshu.io/upload_images/5649240-004d715f5482afdc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如果是WEB开发，对外声明接口，自己业务去实现接口，再暴露接口去调用具体业务的实现。

如上图所示，购物车模块对外提供两个服务：getCartCount 和 addToCart ，产品页面需要展示当前用户购物车的数量，以及把产品添加到购物车，那产品模块因为依赖了 ServiceCart，可以得到CartService的接口声明定义。

但怎么获取CartService的实例呢？

我相信CartService的实例一定存在世界某个地方，等我去发现。

现在模块化架构就变成如下图：

![增加Service层](http://upload-images.jianshu.io/upload_images/5649240-c0a5e18588ffcb24.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如上图，我们增加了一个Service层，这个Service层虽然画在了Common层上面，但并不依赖Common。

此时，我们要解决的就是 CartService 之我要找到你了！

#### 四、CartService 之我要找到你

如果是服务端开发的话，会把CartService的实现集中注册到 Dubbo 或者 Spring中，通过某个Key得到一个Service的实例。

通过一个Key获得一个实例

前面路由的方案是：通过一个Key获取到一个路径。有点意思。经过比较，ARouter 和 自己通过AIDL来实现两套方案，对比如下：

![选型标准](http://upload-images.jianshu.io/upload_images/5649240-9c3191a82f137d7a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

ARouter 使用运行时注解强大的功能，自建Map把 ```path```  和 Service具体实例建立关联，跨模块可以轻轻松松地按照自定义路径取出实例调用。一项项来说明：

* 稳定性：大厂出品，稳定性都OK。
* 通用技术：第三方注解自建维护Map 和 Android AIDL服务调用，AIDL更通用。
* 跨进程： ARouter 不支持，而AIDL专为跨进程为生。
* 易适配：第三方App接入学习成本而言，ARouter还要学习下下，AIDL是Android开发必备技能，不用二次学习。
* 第三方App调用：系统其他App调用服务定义，ARouter不支持，AIDL 通过Service export= true 的情况下，还是可以支持的。

所以，我们考虑用AIDL Service的方式来实现。

这个时候，你怒了，哥，我裤子都脱了，你就给我看这个？

快 Show Me The Fucking Code! 

#### 五、代码秀

有请24位漂亮的女生登场！

![漂亮女生登场](http://upload-images.jianshu.io/upload_images/5649240-fe8bf540f590f0ae.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

模块结构如下图：
![示例代码结构截图](http://upload-images.jianshu.io/upload_images/5649240-cef204716254957a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

app:  客户端启动入口
module_product : 产品模块
module_shopcart: 购物车模块
service_shopcart: 购物车对外开放服务，通过上面截图可以看到已通过AIDL 声明了 ```IShopcartService``` 接口，并声明了 ```getCartCount ``` 和 ```addToCart(String productId)``` 的能力。

看接口的实现如下：

![AIDL的实现](http://upload-images.jianshu.io/upload_images/5649240-011882f9fce25966.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

上面的是不是最AIDL经典实现？还是原来的配方！还是原来的味道！红茶，我只要王老吉？

并在 ```AndroidManifest.xml``` 中声明Service如下：

![屏幕快照 2017-12-01 17.53.37.png](http://upload-images.jianshu.io/upload_images/5649240-5cbb0aed882a00a8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

同时，注明```<intent-filter>``` 为:```com.ssevening.www.service.shopcart.IShopcartService```，这个就是查找这个Service的Key，也是```IShopcartService``` 的类路径。 在 ```module_product``` 中的 ```ProductActivity``` 中如下方法调用:

![跨模块调用服务](http://upload-images.jianshu.io/upload_images/5649240-68804ba3d66c7946.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```bindService``` 和 ```serviceConnection``` 还是熟悉的味道，但最上面的Intent 本应该 ```intent = new Intent(this,ShopcartService.class)``` , 但```ShopcartService```类定义在 ```module_shopcart```里面，所以我们新增了：```getServiceIntent()``` 并把类名传递进去，然后通过```getPackageManager().queryIntentServices(intent, 0);``` 查询出```action=com.ssevening.www.service.shopcart.IShopcartService```  的```Service```，取第一个封装Intent对象，就可以正常调用AIDL服务了，还是原来的配方，还是原来的味道！

运行截图如下：

![运行截图](http://upload-images.jianshu.io/upload_images/5649240-379c9ffb2a4d790a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

购物车数量和添加到购物车的结果都已显示出来，同时通过网络调用把 ```www.baidu.com``` 源码也成功返回。

至此，模块间调用基本方案敲定，余下的就是```getServiceIntent()``` 方法的增强优化和下沉抽到Maven仓库的工作了。

最后，上次路由被坑了一次，详见：[Android M queryIntentActivities return null list 蹲坑记](http://www.jianshu.com/p/daebc8461f35) ，用户在App管理界面，可以关掉AppLinks的事，我还记得，所以这次特别去看一下  ```context.getPackageManager().queryIntentServices(intent, 0);```

代码截图如下：

![queryIntentServices](http://upload-images.jianshu.io/upload_images/5649240-d849a4915ed91be6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)，通过看源码，我们发现并没有用户关掉相关的过滤选项，这样我们就放心了。

我们接着说模块间事件传递。 比如 登陆状态的变化；在金币频道点击签到按钮，跳转到签到模块签到，签到完成后，回到金币模块签到成功的事情传递。


#### 六、模块间事件传递

事件嘛，那肯定要对比 EventBus 和 广播，对比如下：

![EventBus 和 广播对比](http://upload-images.jianshu.io/upload_images/5649240-168ed194fd977486.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这样一对比，不用接第三方库就能实现，并且是Android通用方案，后期第三方接入和使用都方便，所以模块间事件传递就用：```BroadcastReceiver```，这个连例子都不用写了。大家都懂的。

最后，贴一下代码地址：[Android 模块间通信调用示例代码](https://github.com/ssevening/AndroidAidlExample)


欢迎关注作者微信公众号，及时获得作者更新：

![安卓那些事](http://upload-images.jianshu.io/upload_images/5649240-a68ee0852da1e430.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)































