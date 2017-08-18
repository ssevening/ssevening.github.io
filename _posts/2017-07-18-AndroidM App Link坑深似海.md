---
title: Android M queryIntentActivities return null list 蹲坑记
category: Android开发
feature_image: "https://raw.githubusercontent.com/ssevening/ssevening.github.io/master/assets/android.png"
image: "https://raw.githubusercontent.com/ssevening/ssevening.github.io/master/assets/android.png"
---

Android M queryIntentActivities return null list 蹲坑记

<!-- more -->

### 一、背景

近期在研究路由，详见 [App路由方案选型](http://www.ssevening.com/android%E5%BC%80%E5%8F%91/2017/06/11/Android%E8%B7%AF%E7%94%B1%E6%96%B9%E6%A1%88%E9%80%89%E5%9E%8B/) 然后核心技术就是：


```
// 1. manifest.xml 声明ACTION = VIEW 的intent-filter.

<activity android:name="ProductActivity">
            <intent-filter>
                <action android:name="android.intent.action.VIEW" />

                <category android:name="android.intent.category.DEFAULT" />
                <category android:name="android.intent.category.BROWSABLE" />

                <data
                    android:host="www.ssevening.com"
                    android:path="/product.html"
                    android:scheme="https" />
            </intent-filter>
        </activity>
// java 代码中创建
Intent intent = new Intent(Intent.ACTION_VIEW);
List<ResolveInfo> list = PackageManager.queryIntentActivities(intent, PackageManager.MATCH_DEFAULT_ONLY);
if(list!=null){
	// do route thing
}

```
但是上线后，就有 Android 6.0、7.0 用户反馈应用打不开。然后就觉得诡异了，
明明已经声明了相应的 android.intent.action.VIEW ，但为什么竟然会查询不到列表呢？

### 二、问题排查

毫无头绪的时候，就去Google 搜索，然后发现了这篇文章：[intent resolving in android M](https://medium.com/google-developer-experts/intent-resolving-in-android-m-c17d39d27048)，文章中说，App 6.0 以上引入了APP LINK。然后文章作者开发的一个应用，本来是调用queryIntentActivities 查询谁可以处理Intent了，现在只返回了开通App link的应用。导致作者很不爽，然后写了这篇文章。

然后，我就试着去玩APP_LINK，按如下方式，关掉了APP LINK。
![close_app_link.gif](http://upload-images.jianshu.io/upload_images/5649240-7d64af39e9b79422.gif?imageMogr2/auto-orient/strip)

关掉App link后，发现```queryIntentActivities```无法返回manifest.xml对应的Activity信息了。

然后去看：```queryIntentActivities```的源码，发现红色部分，如果未开始App links，那么就不会添加到返回到边中。

![queryIntentActivities.png](http://upload-images.jianshu.io/upload_images/5649240-876ef9c374866116.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我了个擦，那这个路由方案不就失败了吗？
搞的我心理阴暗了一上午，统结啊，都怀疑人生了！
明明说好的，一套路由规则，简单，易用，兼容App link等方向到底是对还是不对？然后难道真的要用ARouter? 难怪老外要怪Google了，说Google不厚道，是啊，明明是要推广APP LINK，如果开发者按APPLINK来路由，那后面大有可为！偏偏又搞了个开关，这不是Google给我掘下的天坑吗？

苍天！让我哭一会吧！


### 三、振奋一下，总不能一直蹲在坑里面

当没有思路的时候，大多就会去参考一下业内的App，比如淘宝，微信，QQ等。然后发现下面的界面：

![淘宝竟然不能关applink.png](http://upload-images.jianshu.io/upload_images/5649240-39f68ab0dce9b2e8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![淘宝竟然是浏览器.png](http://upload-images.jianshu.io/upload_images/5649240-453b079fdf062464.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

为了弄个清楚，就去看了Settings.apk 的代码。


![setting竟然浏览器就不支持关掉applink.png](http://upload-images.jianshu.io/upload_images/5649240-6cf2e49e96b21f92.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

上面的代码，```mIsBrowser``` 如果是true，APP LINKS选项就给关掉了。
上面的淘宝图片也说明，淘宝果然是把自己做成了浏览器。

因为做成了浏览器，所以用户就不可能给关掉，然后就可以避免```queryIntentActivities```返回不了Activity信息的问题。

那怎样才能把自己也添加到浏览器列表呢？也考虑不给用户关掉的机会，来继续用我们的路由方案。通过反编译UC浏览器的```manifest.xml```文件，看到如下代码：

```
<activity android:theme="@*android:style/Theme.Translucent" android:label="@string/app_name" android:name="com.UCMobile.main.UCMobile" android:finishOnTaskLaunch="true" android:launchMode="singleTask" android:configChanges="mcc|mnc|locale|keyboard|keyboardHidden|orientation|screenLayout|screenSize|smallestScreenSize|layoutDirection|fontScale" android:alwaysRetainTaskState="true" android:windowSoftInputMode="adjustPan|adjustNothing" android:noHistory="true" android:resizeableActivity="false">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
            <intent-filter android:label="@string/open_name">
                <action android:name="android.intent.action.VIEW" />
                <category android:name="android.intent.category.DEFAULT" />
                <category android:name="android.intent.category.BROWSABLE" />
            </intent-filter>
           <!--声明 http 和 https 的 intent-filter 然后不写hosts，即可以把自己变成浏览器-->
            <intent-filter android:label="@string/open_name">
                <action android:name="android.intent.action.VIEW" />
                <category android:name="android.intent.category.DEFAULT" />
                <category android:name="android.intent.category.BROWSABLE" />
                <data android:scheme="http" />
                <data android:scheme="https" />
                <data android:scheme="about" />
                <data android:scheme="ucweb" />
                <data android:scheme="javascript" />
            </intent-filter>
        </activity>

```
通过把自己变成浏览器的方式，成功解决问题。但，内心还是有点淡淡的忧伤！难道我也要向淘宝一样，这样耍一回流氓吗？我应该是个有原则的人。那有没有更好的方案呢？

### 四、除浏览器外的方案

然后继续思考，App link 和 路由用到的都是 ```Intent.ACTION_VIEW```，而用户在设置界面，关掉的，也是 ```Intent.ACTION_VIEW``` 的开关。那我们可以为路由重建一个```NAV_ACTION_VIEW``` 。形如如下代码：

```
        <activity android:name="ProductActivity">
            <intent-filter>
                <action android:name="android.intent.action.VIEW" />
                <!--添加我们自己的路由ACTION-->
                <action android:name="android.intent.action.NAV.VIEW.ACTION" />

                <category android:name="android.intent.category.DEFAULT" />
                <category android:name="android.intent.category.BROWSABLE" />
                <data
                    android:host="www.ssevening.com"
                    android:path="/product.html"
                    android:scheme="https" />
            </intent-filter>
        </activity>

// 这样再查询对应的ACTION，妈妈就再也不用担心AndroidM的用户会关掉applink了
Intent intent = new Intent("android.intent.action.NAV.VIEW.ACTION");
List<ResolveInfo> list = PackageManager.queryIntentActivities(intent, PackageManager.MATCH_DEFAULT_ONLY);

```

至此，终于从坑里面走出来了！

至此，Android的路由方案也就确定了。

最后来总结一下 intent-filter路由的事情。

* 什么是路由

![美丽的千岛湖.jpeg](http://upload-images.jianshu.io/upload_images/5649240-cb65644309f79279.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
千岛湖有众多岛屿（神龙岛、龙山岛等），就类似于N多个App和N多个网站。而在岛屿内，同样也会有不同的景点(APP页面)，通过岛的名称找到对应的岛上的位置，就是路由要完成的事情。

* intent-filter路由的横向对比

![路由方案对比.png](http://upload-images.jianshu.io/upload_images/5649240-b8bcf9967c29e08e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

因为在App上是长不出亚马逊森林的，但在WEB上可以，所以我们的路由方案，就是要和WEB建立联系，建立关系，向WEB要流量，和WEB强绑定，然后基于WEB的雄厚实力，让App飞得更高！














