---
title: OkHttp-主流程分析
categories: Tools
tags: 源码分析
keywords: 源码分析 OkHttp
description: OkHttp源码主流程分析
abbrlink: 616e6cce
date: 2019-09-06 09:22:19
---
### 三个问题
1. okhttp发送一次请求的完整流程
2. okhttp的线程调度机制
3. okhttp的连接复用机制

#### okhttp发送一次请求的完整流程
从一次最基本的网络请求看Okhttp做了什么
```java
Request request = new Request.Builder()
        .url("http://127.0.0.1:8080")
        .build();
OkHttpClient client = new OkHttpClient()
        .newBuilder()
        .build();
client.newCall(request).enqueue(new Callback() {
    @Override
    public void onFailure(Call call, IOException e) {
        
    }

    @Override
    public void onResponse(Call call, Response response) throws IOException {

    }
});
//Response response = client.newCall(request).execute()
```
一次基本的请求涉及到四个类
- OkhttpClient
- Request
- Call
- Response  

下面分别对他们进行分析
##### OkhttpClient
```java
public class OkHttpClient implements Cloneable, Call.Factory, WebSocket.Factory{

    public OkHttpClient() {
        this(new Builder());
    }
  
    OkHttpClient(Builder builder) {
        //builder 中参数赋值给client
        ...
    }

    Internal.instance = new Internal(){
        ...
    }
    final Dispatcher dispatcher;
    final @Nullable Proxy proxy;
    final List<Protocol> protocols;
    final List<ConnectionSpec> connectionSpecs;
    final List<Interceptor> interceptors;
    final List<Interceptor> networkInterceptors;
    final EventListener.Factory eventListenerFactory;
    final ProxySelector proxySelector;
    final CookieJar cookieJar;
    final @Nullable Cache cache;
    final @Nullable InternalCache internalCache;
    final SocketFactory socketFactory;
    final SSLSocketFactory sslSocketFactory;
    final CertificateChainCleaner certificateChainCleaner;
    final HostnameVerifier hostnameVerifier;
    final CertificatePinner certificatePinner;
    final Authenticator proxyAuthenticator;
    final Authenticator authenticator;
    final ConnectionPool connectionPool;
    final Dns dns;
    
    @Override public Call newCall(Request request) {
        return RealCall.newRealCall(this, request, false /* for web socket */);
    }
    
    public WebSocket newWebSocket(Request request, WebSocketListener listener){...}
  
    public static final class Builder {
        ...
        public Builder(){}
        Builder(OkHttpClient okHttpClient){}
        public OkHttpClient build() {
            return new OkHttpClient(this);
        }
    }
}
```
从OkhttpClient实现的两个接口`Call.Factory`和`WebSocket.Factory`可以看出是一个工厂类，提供`Call`和`Websocket`的创建方法。因为这个工厂的属性较多(包括代理、连接池、调度器、CookieJar等客户端相关)，构造较为复杂，因此以构造者模式来构建，即`OkhttpClient.Builder`。

##### Request
```java
public final class Request{
    Request(Builder builder) {
        ...
    }
    
    final HttpUrl url;
    final String method;
    final Headers headers;
    final @Nullable RequestBody body;
    final Map<Class<?>, Object> tags;
    private volatile @Nullable CacheControl cacheControl;
    ...
    
    public static class Builder {...}
}
```
可以看出，`Request`是对请求的封装，包含url、method、请求头、请求体等，同样提供build构造方式。

##### Call
`Call`是一个接口，其实现类为RealCall
```java
final class RealCall implements Call {

    private RealCall(OkHttpClient client, Request originalRequest, boolean forWebSocket) {...}
    
    final OkHttpClient client;
    private Transmitter transmitter;
    final Request originalRequest;
    
    static RealCall newRealCall(OkHttpClient client, Request originalRequest, boolean forWebSocket) {...}
    
    //同步发送请求
    public Response execute(){...}
    
    //异步发送请求
    public void enqueue(Callback responseCallback){...}
    
    final class AsyncCall extends NamedRunnable{
        private final Callback responseCallback;
        private volatile AtomicInteger callsPerHost = new AtomicInteger(0);
        ...
    }
    
    //真正执行责任链，发送请求的方法
    Response getResponseWithInterceptorChain() throws IOException {...}
}
```
`RealCall`的构造只能通过OkhttpClient工厂或`clone()`方法来创建，是对一次请求的封装  
`RealCall.AsyncCall`则是`RealCall`的异步模式，继承自`NamedRunnable`，本质是一个`Runnable`，是Dispatcher调度的对象。  

在得到`Call`对象之后，可以通过`execute()`、`enqueue(Callback responseCallback)`发送一次网络请求
```java
@Override public Response execute() throws IOException {
    synchronized (this) {
      if (executed) throw new IllegalStateException("Already Executed");
      executed = true;
    }
    transmitter.timeoutEnter();
    transmitter.callStart();
    try {
        client.dispatcher().executed(this);
        //返回网络请求的结果
        return getResponseWithInterceptorChain();
    } finally {
        client.dispatcher().finished(this);
    }
}

@Override public void enqueue(Callback responseCallback) {
    synchronized (this) {
      if (executed) throw new IllegalStateException("Already Executed");
      executed = true;
    }
    transmitter.callStart();
    //将AsyncCall传递给dispatcher调度并执行，最终会执行RealCall.AsyncCall.execute()方法
    client.dispatcher().enqueue(new AsyncCall(responseCallback));
}

RealCall.AsyncCall {
    @Override protected void execute() {
        boolean signalledCallback = false;
        transmitter.timeoutEnter();
        try {
        //得到网络返回的结果
        Response response = getResponseWithInterceptorChain();
        signalledCallback = true;
        //异步回调给上层
        responseCallback.onResponse(RealCall.this, response);
        } catch (IOException e) {
        if (signalledCallback) {
          // Do not signal the callback twice!
          Platform.get().log(INFO, "Callback failure for " + toLoggableString(), e);
        } else {
          responseCallback.onFailure(RealCall.this, e);
        }
        } finally {
        client.dispatcher().finished(this);
        }
    }
  }
}
```
其中执行涉及到的`Dispatcher`会在稍后分析，可以看到，不管是调用`execute`还是`enqueue`，都是通过执行`RealCall.getResponseWithInterceptorChain()`得到`Response`。
```java
  Response getResponseWithInterceptorChain() throws IOException {
    // Build a full stack of interceptors.
    List<Interceptor> interceptors = new ArrayList<>();
    interceptors.addAll(client.interceptors());
    interceptors.add(new RetryAndFollowUpInterceptor(client));
    interceptors.add(new BridgeInterceptor(client.cookieJar()));
    interceptors.add(new CacheInterceptor(client.internalCache()));
    interceptors.add(new ConnectInterceptor(client));
    if (!forWebSocket) {
      interceptors.addAll(client.networkInterceptors());
    }
    interceptors.add(new CallServerInterceptor(forWebSocket));

    Interceptor.Chain chain = new RealInterceptorChain(interceptors, transmitter, null, 0,
        originalRequest, this, client.connectTimeoutMillis(),
        client.readTimeoutMillis(), client.writeTimeoutMillis());

    boolean calledNoMoreExchanges = false;
      Response response = chain.proceed(originalRequest);
      if (transmitter.isCanceled()) {
        closeQuietly(response);
        throw new IOException("Canceled");
      }
      return response;
  }
```
通过责任链模式的设计，封装为`RealInterceptorChain`，整个链条依次为
- client.interceptors //上层自定义的Interceptor
- RetryAndFollowUpInterceptor //重试和重定向相关的Interceptor
- BridgeInterceptor //将user request转换为network request
- CacheInterceptor //缓存Response及从缓存中读取response
- ConnectInterceptor //打开连接
- networkInterceptors //上层自定义的网络拦截器
- CallServerInterceptor //最后一个拦截器 真正和服务器交互，并得到response  

至此，可以得到okhttp进行一次网络请求大概进行了以下步骤  
<img src="http://ww1.sinaimg.cn/mw690/0077sXesly1g4b1kizq1wj30hv1iw787.jpg" width="200" align=center />

#### 线程调度机制
通过RealCall的源码可以看到同步和异步调用最后都会调用client.dispatcher()的`executed()`和`enqueue()`，下面我们看一下Dispatcher的源码里都做了些什么。
```java
public final class Dispatcher{
    //最大请求数
    private int maxRequests = 64;
    //每个Host的最大请求数
    private int maxRequestsPerHost = 5;

    private final Deque<AsyncCall> readyAsyncCalls = new ArrayDeque<>();

    private final Deque<AsyncCall> runningAsyncCalls = new ArrayDeque<>();

    private final Deque<RealCall> runningSyncCalls = new ArrayDeque<>();
    
    //获取执行线程池
    public synchronized ExecutorService executorService() {
        if (executorService == null) {
            executorService = new ThreadPoolExecutor(0, Integer.MAX_VALUE, 60, TimeUnit.SECONDS,
            new SynchronousQueue<>(), Util.threadFactory("OkHttp Dispatcher", false));
        }
        return executorService;
    }
    ...
    //异步执行网络请求
    void enqueue(AsyncCall call) {
        synchronized (this) {
          readyAsyncCalls.add(call);
    
          // Mutate the AsyncCall so that it shares the AtomicInteger of an existing running call to
          // the same host.
          if (!call.get().forWebSocket) {
            AsyncCall existingCall = findExistingCallWithHost(call.host());
            if (existingCall != null) call.reuseCallsPerHostFrom(existingCall);
          }
        }
        promoteAndExecute();
    }
    ...
    //同步执行请求
    synchronized void executed(RealCall call) {
        //仅添加至队列中，不做其他处理
        runningSyncCalls.add(call);
    }
    ...
}
```
`Dispatcher`内部维护一个大小为Integer.MAX_VALUE的线程池，同时使用三个双端队列分别记录不同状态的Call。  
执行`executed()`时，仅将Call添加至`runningSyncCalls`队列中，仅做记录用。
执行`enqueue()`时，添加至`readyAsyncCalls`中，同时判断
#### 连接复用机制

#### 参考链接
1. [郭孝星开源分析项目](https://github.com/sucese/android-open-framework-analysis)
2. [Okhttp源码分析](https://www.jianshu.com/p/37e26f4ea57b)