---
title: Android逆向之Apk加固原理解析
top: false
cover: false
toc: true
mathjax: true
sticky: 1
date: 2022-01-17 17:27:25
password:
summary:
tags: [逆向, android]
categories: [Android]
---



<div align="center"><iframe width="560" height="315" src="https://www.youtube.com/embed/Oa49Nym7wJk" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></div>



### Android App 启动流程

先用一张图片总体概括下App进程的创建流程：

![img](http://img.heshipeng.com/202201171911278.png)

当点击桌面`app`的时候，发起进程`Launcher`调用`startActivity/startService`等方法通过`binder`通信发送消息给`system_server`进程，告诉`system_server`启动哪一个`app`，`system_server`调用`Process.start(android.app.ActivityThread)`，而后通过`socket`通信告知`Zygote`进程去`fork`子进程，即`app`进程，进程创建后将`ActivityThread`加载进去，执行`ActivityThread.main()`方法。看下`ActivityThread`这个类：

```java
public final class ActivityThread extends ClientTransactionHandler {
    /** Reference to singleton {@link ActivityThread} */
    private static volatile ActivityThread sCurrentActivityThread;
  
    public static ActivityThread currentActivityThread() {
        return sCurrentActivityThread;
    }
  
    public static void main(String[] args) {
        Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "ActivityThreadMain");

        // CloseGuard defaults to true and can be quite spammy.  We
        // disable it here, but selectively enable it later (via
        // StrictMode) on debug builds, but using DropBox, not logs.
        CloseGuard.setEnabled(false);

        Environment.initForCurrentUser();

        // Set the reporter for event logging in libcore
        EventLogger.setReporter(new EventLoggingReporter());

        // Make sure TrustedCertificateStore looks in the right place for CA certificates
        final File configDir = Environment.getUserConfigDirectory(UserHandle.myUserId());
        TrustedCertificateStore.setDefaultUserDirectory(configDir);

        Process.setArgV0("<pre-initialized>");

        Looper.prepareMainLooper();

        // Find the value for {@link #PROC_START_SEQ_IDENT} if provided on the command line.
        // It will be in the format "seq=114"
        long startSeq = 0;
        if (args != null) {
            for (int i = args.length - 1; i >= 0; --i) {
                if (args[i] != null && args[i].startsWith(PROC_START_SEQ_IDENT)) {
                    startSeq = Long.parseLong(
                            args[i].substring(PROC_START_SEQ_IDENT.length()));
                }
            }
        }
        ActivityThread thread = new ActivityThread();
        thread.attach(false, startSeq);

        if (sMainThreadHandler == null) {
            sMainThreadHandler = thread.getHandler();
        }

        if (false) {
            Looper.myLooper().setMessageLogging(new
                    LogPrinter(Log.DEBUG, "ActivityThread"));
        }

        // End of event ActivityThreadMain.
        Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
        Looper.loop();

        throw new RuntimeException("Main thread loop unexpectedly exited");
    }
}
```

`ActivityThread`是一个单例类，其中的`sCurrentActivityThread`静态变量用于全局保存创建的`ActivityThread`实例，同时还提供了`public static ActivityThread currentActivityThread()`静态函数用于获取当前虚拟机创建的`ActivityThread`实例。`ActivityThread.main()`函数是`java`中的入口`main`函数，这里会启动主消息循环，并创建`ActivityThread`实例，之后调用`thread.attach(false)`完成一系列初始化准备工作，并完成全局静态变量`sCurrentActivityThread`的初始化。之后主线程进入消息循环，等待接收来自系统的消息。当收到系统发送来的`bindApplication`的进程间调用时，调用函数`handleBindApplication`来处理该请求。



看下`handleBindApplication`主要逻辑：

```java
private void handleBindApplication(AppBindData data) {
    //step 1: 创建LoadedApk对象
    data.info = getPackageInfoNoCheck(data.appInfo, data.compatInfo);
    ...
    //step 2: 创建ContextImpl对象;
    final ContextImpl appContext = ContextImpl.createAppContext(this, data.info);
 
    //step 3: 创建Instrumentation
    mInstrumentation = new Instrumentation();
 
    //step 4: 创建Application对象;在makeApplication函数中调用了newApplication，在该函数中又调用了app.attach(context)
    //在attach函数中调用了  Application.attachBaseContext函数
    Application app = data.info.makeApplication(data.restrictedBackupMode, null);
    mInitialApplication = app;
 
    //step 5: 安装providers
    List<ProviderInfo> providers = data.providers;
    installContentProviders(app, providers);
 
    //step 6: 执行Application.Create回调
    mInstrumentation.callApplicationOnCreate(app);
```

在 `handleBindApplication`函数中第一次进入了`app`的代码世界，该函数功能是启动一个`application`，并把系统收集的`apk`组件等相关信息绑定到`application`里，在创建完`application`对象后，接着调用了`application`的`attachBaseContext`方法，之后调用了`application`的`onCreate`函数。由此可以发现，`app`的`Application`类中的`attachBaseContext`和`onCreate`这两个函数是最先获取执行权进行代码执行的。这也是为什么各家的加固工具的主要逻辑都是通过替换`app`入口`Application`，并自实现这两个函数，在这两个函数中进行代码的脱壳以及执行权交付的原因。



站在`ClassLoader`的角度看整个`App`加载的流程如下：

![image-20220118141628602](http://img.heshipeng.com/202201181416237.png)

如果一个`app`没有加壳，在第二步`PathClassLoader`自然而然的加载的是`app`所有的类信息，如果加壳了，则`PathClassLoader`加载的只有壳的代码，当前还没有加载`app` 真正的代码，也就是壳解密后释放的代码。



### App加固原理

通过前面`app`运行流程的了解可以知道`app`最先获得执行权限的是`app`中声明的`Application`类中的`attachBaseContext`和`onCreate`函数。因此，壳要想完成应用中加固代码的解密以及应用执行权的交付就都是在这两个函数上做文章。下面这张图大致讲了加壳应用的运行流程。

![下载](http://img.heshipeng.com/202201181444463.png)

当壳在函数`attachBaseContext`和`onCreate`中执行完加密的`dex`文件的解密后，通过自定义的`Classloader`在内存中加载解密后的`dex`文件。为了解决后续应用在加载执行解密后的`dex`文件中的`Class`和`Method`的问题，接下来就是通过利用`java`的反射修复一系列的变量。其中最为重要的一个变量就是应用运行中的`Classloader`，只有`Classloader`被修正后，应用才能够正常的加载并调用`dex`中的类和方法，否则的话由于`Classloader`的双亲委派机制，最终会报`ClassNotFound`异常，应用崩溃退出。



