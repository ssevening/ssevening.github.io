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
* JSON to POJO 是在IO线程中执行的。而volley是在主线程中完成的。这点性能上要好一些。

下面来爆菊：
* 上面代码中，必须添加 baseUrl，这就是他的致命问题，比如我们默认访问 hangzhou.api.com 但当杭州出现问题的时候，我们要切换到 shanghai.api.com,这种情况下，baseUrl就显得有点尴尬了。当然，你说我们可以解析API，然后动态构建Retrofit，那下面我再举例子。
* 对于那种API提供是多方提供的，如果有一个模块的展示数据源，今天是 hot.api.com/product.json 明天是 cold.api.com/p.json 这种情况，那也就费了。
* 或者还有一种解决方案：就是定义一个通用的Service,即如下代码的情况。但这就一点也不符合Retrofit的设计了。

```
// parse url get base url and url path: http://api.com/aaa.htm
String baseUrl = "http://api.com";
String path = "aaa.htm";
Retrofit retrofit = new Retrofit.Builder()
    .baseUrl(baseUrl)
    .addConverterFactory(GsonConverterFactory.create())
    .build();
    
Service的代码实现如下：
    @GET("{path}")
    public Observable<JSON> getApiData(@Path("path") String path);    

调用的时候，如下面方法的调用
String json = service.getApiData(path);
// 下面再把json解析成pojo.好吧。我只能帮你到这里了。


```

所以综上所述，对于集中式管理API的App来说，使用Retrofit是个不错的选择，特别是结合RXJAVA，但当App越来越大的时候，特别是涉及到容灾，域名切换，甚至多机房的情况下，使用起来就费了。

* 再说一下实现原理：先说入口：GitHubService service = retrofit.create(GitHubService.class); Create方法为入口。

```
public <T> T create(final Class<T> service) {
    // 传递过来的类进行校验，看看不是接口类，是不是没有接口定义之类的。
    Utils.validateServiceInterface(service);
    if (validateEagerly) {
      eagerlyValidateMethods(service);
    }
    // 这里就是动态代理了
    return (T) Proxy.newProxyInstance(service.getClassLoader(), new Class<?>[] { service },
        new InvocationHandler() {
          // 看看是什么操作平台，如Android JAVA8之类的，根据平台最终实现的方案不同。我们这里直接跳到最终的 loadMethodHandler方法
          private final Platform platform = Platform.get();

          @Override public Object invoke(Object proxy, Method method, Object... args)
              throws Throwable {
            // If the method is a method from Object then defer to normal invocation.
            if (method.getDeclaringClass() == Object.class) {
              return method.invoke(this, args);
            }
            if (platform.isDefaultMethod(method)) {
              return platform.invokeDefaultMethod(method, service, proxy, args);
            }
            return loadMethodHandler(method).invoke(args);
          }
        });
  }
```
下面的方法，我们再看MethodHandler.create(this, method)的方法，这里主要就是先从缓存中取一下没有这个实例，如果有，直接用，如果没有，就创建，所以第二次调用会快，第一次调用会慢一丢丢。上面的代码中，最后就是搞出一个MethodHandler来后，再调用相应的invoke方法。

```
  MethodHandler loadMethodHandler(Method method) {
    MethodHandler handler;
    synchronized (methodHandlerCache) {
      // 这里是用来缓存MethodHandler的。
      handler = methodHandlerCache.get(method);
      if (handler == null) {
        handler = MethodHandler.create(this, method);
        methodHandlerCache.put(method, handler);
      }
    }
    return handler;
  }
```

下面的代码中，封装了请求工程，以前数据返回的转换适配器。我们直接找MothodHandler.invoke方法的实现类去。

```
  static MethodHandler create(Retrofit retrofit, Method method) {
    CallAdapter<?> callAdapter = createCallAdapter(method, retrofit);
    Type responseType = callAdapter.responseType();
    if (responseType == Response.class || responseType == okhttp3.Response.class) {
      throw Utils.methodError(method, "'"
          + Types.getRawType(responseType).getName()
          + "' is not a valid response body type. Did you mean ResponseBody?");
    }
    Converter<ResponseBody, ?> responseConverter =
        createResponseConverter(method, retrofit, responseType);
    RequestFactory requestFactory = RequestFactoryParser.parse(method, responseType, retrofit);
    return new MethodHandler(retrofit.callFactory(), requestFactory, callAdapter,
        responseConverter);
  }
```
下面的代码就是invoke的实现类，把所有builder过程中的参数都带了进去。发送网络请求的真实实现马上要来了。继续看：callAdapter.adapt的实现类

```
  Object invoke(Object... args) {
    return callAdapter.adapt(
        new OkHttpCall<>(callFactory, requestFactory, args, responseConverter));
  }

```

我们看到，执行的是RxJAVA的代码，这里的核心是被观察者的生成。我们下一步去看：new CallOnSubscribe的实现。

```

    @Override public <R> Observable<R> adapt(Call<R> call) {
      return Observable.create(new CallOnSubscribe<>(call)) //
          .flatMap(new Func1<Response<R>, Observable<R>>() {
            @Override public Observable<R> call(Response<R> response) {
              if (response.isSuccess()) {
                return Observable.just(response.body());
              }
              return Observable.error(new HttpException(response));
            }
          });
    }
  }
```

然后就有下下面的代码, 在call方法中，call.execute() 最终发送了网络请求，看一下他的具体实现。在一个代码段。

```
  static final class CallOnSubscribe<T> implements Observable.OnSubscribe<Response<T>> {
    private final Call<T> originalCall;

    CallOnSubscribe(Call<T> originalCall) {
      this.originalCall = originalCall;
    }

    @Override public void call(final Subscriber<? super Response<T>> subscriber) {
      // Since Call is a one-shot type, clone it for each new subscriber.
      final Call<T> call = originalCall.clone();

      // Attempt to cancel the call if it is still in-flight on unsubscription.
      subscriber.add(Subscriptions.create(new Action0() {
        @Override public void call() {
          call.cancel();
        }
      }));

      try {
        Response<T> response = call.execute();
        if (!subscriber.isUnsubscribed()) {
          subscriber.onNext(response);
        }
      } catch (Throwable t) {
        Exceptions.throwIfFatal(t);
        if (!subscriber.isUnsubscribed()) {
          subscriber.onError(t);
        }
        return;
      }

      if (!subscriber.isUnsubscribed()) {
        subscriber.onCompleted();
      }
    }
  }
```
实现类okhttp3的代码，发送了网络请求，然后并且解析返回，然后转换为我们要返回的POJO对象，至此就完成了一次网络交互。

```

  @Override public Response<T> execute() throws IOException {
    okhttp3.Call call;

    synchronized (this) {
      if (executed) throw new IllegalStateException("Already executed.");
      executed = true;

      if (creationFailure != null) {
        if (creationFailure instanceof IOException) {
          throw (IOException) creationFailure;
        } else {
          throw (RuntimeException) creationFailure;
        }
      }

      call = rawCall;
      if (call == null) {
        try {
          call = rawCall = createRawCall();
        } catch (IOException | RuntimeException e) {
          creationFailure = e;
          throw e;
        }
      }
    }

    if (canceled) {
      call.cancel();
    }

    return parseResponse(call.execute());
  }

```









欢迎关注作者微信公众号：

![微信公众号](https://ssevening.github.io/assets/weichat_qrcode.jpg)

更欢迎加入作者维护的小密圈：
![微信公众号](https://ssevening.github.io/assets/mi_qrcode.png)










