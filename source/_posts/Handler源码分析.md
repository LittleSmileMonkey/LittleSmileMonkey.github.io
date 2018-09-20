---
title: Handler源码分析（未完）
categories: 源码分析
tags: Handler
keywords: Android Handler 源码分析
abbrlink: 3e1578ca
date: 2018-09-02 22:13:57
description:
---
涉及到四个类：Looper、Handler、MessageQueue 、ActivityThread 
Looper：最核心的类，循环取出消息，ThreadLocal<Looper>来保证Looper和Thread一一对应、并初始化与Thread对应的MessageQueue，没有消息时阻塞在MessageQueue.next()中
```Java
private static void prepare(boolean quitAllowed) {
        if (sThreadLocal.get() != null) {
            throw new RuntimeException("Only one Looper may be created per thread");
        }
        sThreadLocal.set(new Looper(quitAllowed));
    }
```

Handler: 对消息进行调度、处理  
MessageQueue：消息队列，有插入enqueueMessage()和取出next()等方法，消息队列的消息是按待处理时间排序的。  
ActivityThread：app主线程(UI线程)，main函数中会初始化Handler、并调用Looper.loop()。

<!-- more -->

##### 从使用方式看源码  
在开发中我们是这样使用Handler的：
```java

public class MainActivity extends AppCompatActivity {  
  
    public static final int MSG_FINISH = 0X001;  
    //创建一个Handler的匿名内部类  
    private Handler handler = new Handler() {  
  
        @Override  
        public void handleMessage(Message msg) {  
            switch (msg.what) {  
                case MSG_FINISH:  
                    LogUtils.e("handler所在的线程id是-->" + Thread.currentThread().getName());  
                    break;  
            }  
        }  
  
    };  
  
    @Override  
    protected void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.activity_main);  
        //启动耗时操作  
        new Thread() {  
            public void run() {  
                try {  
                    LogUtils.e("耗时子线程的Name是--->" + Thread.currentThread().getName());  
                    //在子线程运行  
                    Thread.sleep(2000);  
                    //完成后，发送下载完成消息  
                    handler.sendEmptyMessage(MSG_FINISH);  
                } catch (InterruptedException e) {  
                    e.printStackTrace();  
                }  
            }  
        }.start(); 
    }  
} 

```
首先，程序会初始化Handler，那么new Handler()做了什么呢？


```java

public class Handler{
    ...
    
    public Handler() {
        this(null, false);
    }
    ...
    
    public Handler(Callback callback, boolean async) {
        if (FIND_POTENTIAL_LEAKS) {
            final Class<? extends Handler> klass = getClass();
            if ((klass.isAnonymousClass() || klass.isMemberClass() || klass.isLocalClass()) &&
                    (klass.getModifiers() & Modifier.STATIC) == 0) {
                Log.w(TAG, "The following Handler class should be static or leaks might occur: " +
                    klass.getCanonicalName());
            }
        }

        mLooper = Looper.myLooper();
        if (mLooper == null) {
            throw new RuntimeException(
                "Can't create handler inside thread that has not called Looper.prepare()");
        }
        mQueue = mLooper.mQueue;
        mCallback = callback;
        mAsynchronous = async;
    }
}

```
最终调用了构造函数Handler(Callback callback, boolean async)，并且在构造函数中调用了Looper.myLooper()，我们再来看看Looper中的源码：
```
    static final ThreadLocal<Looper> sThreadLocal = new ThreadLocal<Looper>();

/**
 * Return the Looper object associated with the current thread.  Returns
 * null if the calling thread is not associated with a Looper.
 */
public static @Nullable Looper myLooper() {
    return sThreadLocal.get();
}

```
可以看到，myLooper()会返回当前线程相关的Looper，如果当前线程没有初始化Looper对象则会返回null，然后我们找到Looper的构造函数:

```java
//非主线程初始化Looper时调用的方法
    public static void prepare() {
        prepare(true);
    }

//真正初始化Looper的方法，QuitAllowed表示当前Looper是否允许退出
//只有主线程不允许退出，其他线程的Looper均能退出
private static void prepare(boolean quitAllowed) {
    if (sThreadLocal.get() != null) {
        throw new RuntimeException("Only one Looper may be created per thread");
    }
    sThreadLocal.set(new Looper(quitAllowed));
}

//主线程初始化Looper调用的函数
public static void prepareMainLooper() {
        prepare(false);
        synchronized (Looper.class) {
            if (sMainLooper != null) {
                throw new IllegalStateException("The main Looper has already been prepared.");
            }
            sMainLooper = myLooper();
        }
    }

```
此时我们可以知道Looper的部分特性：
1. ThreadLocal<Looper>说明它是和线程相关联的，每个线程有且只能有一个Looper
2. 
