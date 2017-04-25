---
title: Reservoir 实现的简易缓存
category: Android开发
feature_image: "https://ssevening.github.io/assets/android.png"
image: "https://ssevening.github.io/assets/android.png"
---
Reservoir 实现的简易缓存的业务使用场景 和 Retrofit网络框架评测


<!-- more -->

## 一、Reservoir 实现的简易缓存
* 先贴地址：[GitHub](https://github.com/ssevening/Reservoir)

### 使用场景
* App开发中，希望缓存一些数据对象，比如某些产品数据，在服务端获取后，直接从缓存读取，但又不希望这个缓存太大。如果自己写的话，就要维护缓存大小，缓存数量等内容。
* 其实图片模块的LRU DiskCache，就是本开源组件的实现，只是这里不是用在图片上，而是数据对象。

### 实现原理：
* LRU缓存，JAVA对象转换为JSON字符串写入。详见 put方法:

```
    public static void put(final String key, final Object object) throws IOException {
        failIfNotInitialised();
        String json = sGson.toJson(object);
        cache.put(key, json);
    }
```
* 异步存入实现,直接用AsyncTask了，好吧。这样也可以。

```
private static class PutTask extends AsyncTask<Void, Void, Void> {
        private final String key;
        private Exception e;
        private final ReservoirPutCallback callback;
        final Object object;

        private PutTask(String key, Object object, ReservoirPutCallback callback) {
            this.key = key;
            this.callback = callback;
            this.object = object;
            this.e = null;
        }

        @Override
        protected Void doInBackground(Void... params) {

            try {
                String json = sGson.toJson(object);
                cache.put(key, json);
            } catch (Exception e) {
                this.e = e;
            }
            return null;
        }

        @Override
        protected void onPostExecute(Void aVoid) {
            if (callback != null) {
                if (e == null) {
                    callback.onSuccess();
                } else {
                    callback.onFailure(e);
                }
            }
        }

    }
```

* 我们再来看看读取：从Cache中取出来后，再通过Json转换为对象。如果未获取到数据，会有空指针抛出

```
    public static <T> T get(final String key, final Class<T> classOfT) throws IOException {
        failIfNotInitialised();
        String json = cache.getString(key).getString();
        T value = sGson.fromJson(json, classOfT);
        if (value == null)
            throw new NullPointerException();
        return value;
    }
```

* 我们再看异步读取：还是AsyncTask，这里可以针对性的去优化，增加线程池。

```
    /**
     * AsyncTask to perform get operation in a background thread.
     */
    private static class GetTask<T> extends AsyncTask<Void, Void, T> {
        private final String key;
        private final ReservoirGetCallback callback;
        private final Class<T> classOfT;
        private final Type typeOfT;
        private Exception e;

        private GetTask(String key, Class<T> classOfT, ReservoirGetCallback callback) {
            this.key = key;
            this.callback = callback;
            this.classOfT = classOfT;
            this.typeOfT = null;
            this.e = null;
        }

        private GetTask(String key, Type typeOfT, ReservoirGetCallback callback) {
            this.key = key;
            this.callback = callback;
            this.classOfT = null;
            this.typeOfT = typeOfT;
            this.e = null;
        }

        @Override
        protected T doInBackground(Void... params) {
            try {
                String json = cache.getString(key).getString();
                T value;
                if (classOfT != null) {
                    value = sGson.fromJson(json, classOfT);
                } else {
                    value = sGson.fromJson(json, typeOfT);
                }
                if (value == null) {
                    throw new NullPointerException();
                }
                return value;
            } catch (Exception e) {
                this.e = e;
                return null;
            }
        }

        @Override
        protected void onPostExecute(T object) {
            if (callback != null) {
                if (e == null) {
                    callback.onSuccess(object);
                } else {
                    callback.onFailure(e);
                }
            }
        }

    }

```

### 和SP的区别
* SP是持久化的，App不删除，不卸载，就一直存在，但Reservoir的缓存，达到初始化容量就会清除掉，近期未使用的数据。
* 另外一种缓存的实现思路：基于时间的DB，存取数据时，把过期时间同步存入，然后获取时，比较时间有效性，决定是否用缓存的值。


## 二、客户端架构一些参考文章
* 有赞的：https://youzanmobile.github.io/2017/04/14/youzan-app-modularization/
* 大众点评：http://jiajixin.cn/2016/09/28/android_modularization/


### 三、Retrofit网络框架评测

参考的文档：[点击直达](http://gank.io/post/56e80c2c677659311bed9841)

使用方式我就不多介绍了，大家可以看 [官网](http://square.github.io/retrofit/) 的介绍。反正就是定义接口，然后就可以直接调用了。

```
Retrofit retrofit = new Retrofit.Builder()
    .baseUrl("https://api.github.com")
    .addConverterFactory(GsonConverterFactory.create())
    .build();

GitHubService service = retrofit.create(GitHubService.class);
// 这里就可以直接发送网络请求获取数据了
service.getUserData();

```

先来说一下优点
* 易于开发，接入简单，开发成本低。
* 支持动态参数，动态路径，比如api中有动态的参数的情况。可以在路径中替换。
* RxJAVA无缝支持，rx火了，他也跟着火了。

下面来爆菊：
* 上面代码中，必须添加 baseUrl，这就是他的致命问题，比如我们默认访问 hangzhou.api.com 但当杭州出现问题的时候，我们要切换到 shanghai.api.com,这种情况下，baseUrl就显得有点尴尬了。当然，你说我们可以解析API，然后动态构建Retrofit，那下面我再举例子。
* 对于那种API提供是多方提供的，如果有一个模块的展示数据源，今天是 hot.api.com/product.json 明天是 cold.api.com/p.json 这种情况，那也就费了。

所以综上所述，对于集中式管理API的App来说，使用Retrofit是个不错的选择，特别是结合RXJAVA，但当App越来越大的时候，特别是涉及到容灾，域名切换，甚至多机房的情况下，使用起来就费了。



欢迎关注作者微信公众号

![微信公众号](https://ssevening.github.io/assets/weichat_qrcode.jpg)










