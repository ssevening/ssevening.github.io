---
title: Application onCreate 初始化优化
category: Android开发
feature_image: "https://ssevening.github.io/assets/android.png"
image: "https://ssevening.github.io/assets/android.png"
---

大部分框架为了保证自己的功能全App都可以使用，都强制要求把初始化放到onCreate中，但这样日积月累，Application onCreate简直就是噩梦。一堆的初始化放在那里，性能不断下降。那怎样解决这个问题呢？今天为大家提供一些思路。

<!-- more -->

## 一、异步化
* 可以开线程去初始化的，就通过线程去初始化。如下面的代码

```
new AsyncTask<String,Integer,String>(){

                @Override
                protected String doInBackground(String... params) {
                    // 执行初始化的操作
                    return null;
                }
            }.execute();
```

## 二、按需初始化
* 直接先不初始化，但因为要全局用到，所以用到时，按需加载。详见下方代码:
* 代码核心思想就是当用到的时候，getInstance()的时候，才会执行初始化，前期只是保存了一个Application的引用，等到需要的时候引入。

```
public class LogUtilsInit {

    private static boolean sInitialized;
    private static final Object sLock = new Object();
    private static Context sAppContext;

    private static LogUtils sInstance;

    private LogUtilsInit(Context appContext) {
		 sInstance = new LogUtils(appContext);
    }

    public static void initialize(Context appContext) {
        if (!sInitialized) {
            synchronized (sLock) {
                if (!sInitialized) {
                    sAppContext = appContext;
                    sInitialized = true;
                }
            }
        }
    }

    public static LogUtilsInit getInstance() {
        checkInitialized();
        if (sInstance == null) {
            synchronized (LogUtilsInit.class) {
                if (sInstance == null) {
                    sInstance = new LogUtilsInit(sAppContext);
                }
            }
        }
        return sInstance;
    }

    private static void checkInitialized() {
        if (!sInitialized) {
            throw new Exception("You must init first");
        }
    }

}
```

## 三、其他框架方案
* [App 任务初始化框架 ](https://github.com/ssevening/init)
* 这个就不贴代码了，直接看Github吧。

综上所述，大家可以结合自己项目的情况，选择性的使用相应的优化方案。



欢迎关注作者微信公众号：

![微信公众号](https://ssevening.github.io/assets/weichat_qrcode.jpg)

更欢迎加入作者维护的小密圈：
![微信公众号](https://ssevening.github.io/assets/mi_qrcode.png)



