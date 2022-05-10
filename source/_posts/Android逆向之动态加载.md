---
title: Android逆向之动态加载
top: false
cover: false
toc: true
mathjax: true
date: 2022-01-15 23:40:22
password:
summary:
tags: [逆向, android]
categories: [Android]
---



<div align="middle"><iframe width="560" height="315" src="https://www.youtube.com/embed/k7akz2hdToY" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></div>



### 动态加载

开始正题之前，在这里可以先给动态加载技术做一个简单的定义。真正的动态加载应该是

1. 应用在运行的时候通过加载一些**本地不存在**的可执行文件实现一些特定的功能。
2. 这些可执行文件是**可以替换**的。
3. 更换静态资源(比如换启动图、换主题、或者用服务器参数开关控制广告的隐藏现实等)**不属于**动态加载。
4. `Android`中动态加载的核心思想是动态调用外部的 **`dex`文件**，极端的情况下，`Android APK`自身带有的`Dex`文件只是一个程序的入口(或者说空壳)，所有的功能都通过从服务器下载最新的`Dex`文件完成。



### 两个疑问🤔

提出两个问题，第一，如何在`Android`程序中加载外部`dex`的`class`；第二，对于有生命周期的组件(比如Activity这种类)该如何加载？本文的目的就是通过解决这2个问题从而对`Android`的动态加载技术有一定的了解。



### 类加载器与双亲委派

要解决这俩问题，首先要了解几个概念。



####  类加载器

类加载器顾名思义是用来进行类的加载。分别看下JVM的类加载器和Android的类加载器。



##### JVM的类加载器

JVM的类加载器包括3种：

1. Bootstrap ClassLoader(引导类加载器)

C/C++代码实现的加载器，用于加载指定的JDK的核心类库，比如java.lang，java.util等这些系统类。Java虚拟机的启动就是通过Bootstrap，该Classloader在java里无法获取，负责加载/lib下的类。

2. Extensions ClassLoader(拓展类加载器)

Java中的实现类为ExtClassLoader，提供了除了系统类之外的额外功能，可以在Java里获取，负责加载/lib/ext下的类。

3. Application ClassLoader(应用程序类加载器)

Java中的实现类为AppClassLoader，是与我们接触最多的类加载器，开发人员写的代码默认就是由它来加载，ClassLoader.getSystemCLassLoader返回的就是它。



同时，我们也可以自定义类加载器，只需要通过继承java.lang.ClassLoader类的方式来实现自己的类加载器即可。



##### Android中的类加载器

首先通过一张图，了解各个加载器之间的继承关系。

![image](https://img.heshipeng.com/202201131046618.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



详细看下各个类加载器的作用：

1. ClassLoader为抽象类；
2. BootClassLoader预加载常用类，单例模式。与Java中的BootClassLoader不同，它并不是由C/C++代码实现，而是由Java实现的；
3. BaseDexClassLoader是PathClassLoader, DexClassLoader, InMemoryDexClassLoader的父类，类加载的主要逻辑都是在BaseDexClassLoader完成的；
4. SecureClassLoader继承了抽象类ClassLoader，拓展了ClassLoader类加入了权限方面的功能，加强了安全性，其子类URLClassLoader是用URL路径从jar文件中加载类和资源。

5. PathClassLoader是Android默认使用的类加载器，一个apk中的Activity等类便是在其中加载。

6. DexClassLoader可以加载任意目录下的dex/jar/apk/zip文件，比PathClassLoader更灵活，是实现插件化，热修复以及dex加壳的重点。

7. InMemoryDexClassLoader是8.0引入的，是用于直接从内存中加载dex。

其中重点关注的是：PathClassLoader和DexClassLoader，因为这2个类加载器是我们解决上面2个问题的关键，也是这个动态加载中非常重要的的类加载器。



##### 类加载的时机

1. 隐式加载。

- 创建类的实例。
- 访问类的静态变量，或者为静态变量赋值。
- 调用类的静态方法。
- 使用反射方式来强制创建某个类或者接口对应的java.lang.Class对象。

2. 显式加载。

- 使用LoadClass()加载。
- 使用forName()加载。



##### 类加载的步骤

1. 装载。查找和导入Class文件。

2. 链接。其中解析步骤是可以选择的。

3. - 检查：检查载入的class文件数据的正确性。
   - 准备：给类的静态变量分配存储空间。
   - 解析：将符号引用转换成直接引用。

4. 初始化：即调用<clinit>函数，对静态变量，静态代码块执行初始化工作。

![image](https://img.heshipeng.com/202201131057328.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



##### 编写代码测试Android ClassLoader的继承关系

动手之前先通过[Android源码阅读网站](http://androidxref.com)，看下ClassLoader的源码：



![android ClassLoader类源码](https://img.heshipeng.com/202201131124371.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

打开AndroidXRef，Definition填写`ClassLoader`，搜索包位置`libcore`，如果不确定位置，可以选中全部，只不过搜索出来的结果不比较多，筛选起来麻烦一些。搜索出来一个ClassLoader.java文件，点击进入：

```java
public abstract class ClassLoader {
    // 省略
  
    // The parent class loader for delegation
    // Note: VM hardcoded the offset of this field, thus all new fields
    // must be added *after* it.
    private final ClassLoader parent;
  
  	@CallerSensitive
    public final ClassLoader getParent() {
        return parent;
    }
    
    // 省略 
}    
```

这里关注一个属性和一个成员方法，通过组合关系parent来标识每一个ClassLoader的父亲，这个parent是实现双亲委派的关键，调用getParent成员方法则可以获取到parent属性。



看了这么多理论，没有动手coding去实践，有一点枯燥，接下来编写一个demo去测试一下Android的ClassLoader之间的继承关系。

新建一个项目ClassLoaderTest，然后编码如下：

```java
package com.example.classloadertest;

import androidx.appcompat.app.AppCompatActivity;

import android.content.Context;
import android.os.Bundle;
import android.util.Log;

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        testClassLoader();
    }

    public void testClassLoader() {
        // 获取当前的ClassLoader
        Context context = getApplicationContext();
        ClassLoader thisClassLoader = context.getClassLoader();
        // 或者使用下面这种方式：
        // ClassLoader thisClassLoader = MainActivity.class.getClassLoader();
        Log.i("kanxue", "thisClassLoader:" + thisClassLoader);
        ClassLoader tmpClassLoader = null;
        ClassLoader parentClassLoader = thisClassLoader.getParent();

        // 向上遍历classLoader
        while (parentClassLoader != null) {
            Log.i("kanxue", "this:" + thisClassLoader + ", parent:" + parentClassLoader);
            tmpClassLoader = parentClassLoader.getParent();
            thisClassLoader = parentClassLoader;
            parentClassLoader = tmpClassLoader;
        }
        Log.i("kanxue", "root:" + thisClassLoader);
    }
}
```

代码比较简单，这里也是直接用了上面源码分析的getParent方法，通过getParent方法拿到parent，然后一层层的向上遍历，从而测试出各个ClassLoader的继承关系。

执行结果如下：

![image](https://img.heshipeng.com/202201131142562.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



#### 双亲委派

##### 双亲委派的工作原理

如果一个类加载器收到了类加载请求，它并不会自己先去加载，而是把这个请求委派给父类的加载器去执行，如果父类加载器还存在其父类加载器，则进一步向上委托，依次递归，请求最终将到达顶层的启动类加载器，如果父类加载器可以完成类加载任务，就成功返回，倘若父类加载器无法完成此加载任务，子加载器才会尝试自己去加载，这就是双亲委派模式，即每个儿子都不愿意干活，每次有活就丢给父亲去干，直到父亲说这件事也干不了时，儿子自己想办法去完成，这就是双亲委派。

![image](https://img.heshipeng.com/202201131058935.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



##### 为什么会有双亲委派

1. 避免重复加载，如果已经加载过一次Class，可以直接读取已经加载的Class
2. 更加完全，无法自定义类来代替系统的类，可以防止核心API库被随意篡改。



### 解决第一个问题

通过前面的理论知识，我们知道了DexClassLoader可以加载任意目录下的dex/jar/apk/zip文件，所以我们先来解决第一个问题。



1. 生成一个用来测试的dex文件。

创建项目`DexLoaderTest`，然后新建Class：`TestCLass`。

```java
package com.example.dexloadertest;

import android.util.Log;

public class TestClass {
    public void testFunc() {
        Log.i("kanxue", "call from DexLoaderTest.TestClass.testFunc");
    }
}
```

代码比较简单，只是简单的打印了一条日志。



接着，打包该项目，将生成的apk解压，得到dex文件(如果解压得到多个classes.dex文件，查看下我们编写的TestClass类在哪个classes.dex文件)，然后将这个classes.dex放到手机的内存卡：

```shell
adb push classes.dex /sdcard/
```



2. 加载sdcard上的dex

修改`DexLoaderTest`，修改其`MainActivity`的代码如下：

```java
package com.example.dexloadertest;

import androidx.appcompat.app.AppCompatActivity;
import android.content.Context;
import android.os.Bundle;
import android.os.Environment;

import java.io.File;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;

import dalvik.system.DexClassLoader;

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        String sdcardPath = Environment.getExternalStorageDirectory().getAbsolutePath();
        dexClassLoaderTest(getApplicationContext(), sdcardPath + File.separator + "classes.dex");
    }

    public void dexClassLoaderTest(Context context, String dexFilePath) {
        Class<?> clazz;
        try {
            DexClassLoader dexClassLoader = new DexClassLoader(dexFilePath, null, null, MainActivity.class.getClassLoader());
            clazz = dexClassLoader.loadClass("com.example.dexclass.TestClass");
            if (clazz != null) {
                Method testFuncMethod = clazz.getDeclaredMethod("testFunc");
                Object obj = clazz.newInstance();
                testFuncMethod.invoke(obj);
            }
        } catch (ClassNotFoundException | NoSuchMethodException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (InstantiationException | InvocationTargetException e) {
            e.printStackTrace();
        }
    }
}
```

代码也不复杂，首先实例化一个DexClassLoader对象，然后通过这个对象加载TestClass类，然后拿到testFunc方法，最后通过反射调用这个方法。这里主要关注的是DexClassLoader这个类，我们查看下源码：

```java
public class DexClassLoader extends BaseDexClassLoader {
36    /**
37     * Creates a {@code DexClassLoader} that finds interpreted and native
38     * code.  Interpreted classes are found in a set of DEX files contained
39     * in Jar or APK files.
40     *
41     * <p>The path lists are separated using the character specified by the
42     * {@code path.separator} system property, which defaults to {@code :}.
43     *
44     * @param dexPath the list of jar/apk files containing classes and
45     *     resources, delimited by {@code File.pathSeparator}, which
46     *     defaults to {@code ":"} on Android
47     * @param optimizedDirectory this parameter is deprecated and has no effect since API level 26.
48     * @param librarySearchPath the list of directories containing native
49     *     libraries, delimited by {@code File.pathSeparator}; may be
50     *     {@code null}
51     * @param parent the parent class loader
52     */
53    public DexClassLoader(String dexPath, String optimizedDirectory,
54            String librarySearchPath, ClassLoader parent) {
55        super(dexPath, null, librarySearchPath, parent);
56    }
57}
```

其构造函数需要四个参数，第一个参数是包含资源文件的jar/apk/dex等文件的路径，如果有多个路径，通过:分隔。第二个参数已经废弃。第三个参数为native库的路径，这里我们也是置为null。最后一个是parent类加载器，我们设置为当前类加载器即可。

因为用到对sdcard的读写，所以需要在AndroidManifest.xml中添加相应的权限：

```xml
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" android:maxSdkVersion="28" />
```



运行程序，结果如下：

![image](https://img.heshipeng.com/202201131409909.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



### 解决第二个问题

如果不考虑双亲委派以及Activity生命周期的问题，我们是不是可以用类似于第一个问题的解决方案，采用DexClassLoader加载Activity类，然后使用Intent直接访问这个Activity呢？说干就干。



1. 生成一个用来测试的dex

新建一个TestActivity文件，然后让TestActivity继承AppCompatActivty，并重写onCreate方法。

```java
public class TestActivity extends AppCompatActivity {
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        Log.d("TestActivity", "i am from TestActivity.onCreate");
    }
}
```

这个方法比较简单，仅仅是打印一条日志。

同样地，打包该项目，将生成的apk解压，得到dex文件，然后将这个classes.dex放到手机的内存卡。

```bash
adb push classes3.dex /sdcard/
```



2. 加载并启动Activity

编辑DexClassLoader，修改MainActivity代码如下：

```java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        String sdcardPath = Environment.getExternalStorageDirectory().getAbsolutePath();
        startActivityTest(this, sdcardPath + File.separator + "classes3.dex");
    }

    public void startActivityTest(Context context, String dexFilePath) {
        Class<?> clazz = null;
        DexClassLoader dexClassLoader = new DexClassLoader(dexFilePath, null, null, MainActivity.class.getClassLoader());
        try {
            clazz = dexClassLoader.loadClass("com.example.dexclass.TestActivity");
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
        context.startActivity(new Intent(context, clazz));
    }
}
```



因为有用到`TestActivity`，所以需要在AndroidManifest.xml中声明：

```xml
<activity android:name="com.example.dexclass.TestActivity"/>
```



运行项目，在手机设置中给应用开启sdcard读写权限，然后重新执行，结果报错：

![执行结果](https://img.heshipeng.com/202201131612910.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

想想也不可能成功，要是能跟问题一一样解决，那还叫两个问题吗🤪



那该如何解决第二个问题呢？此时就得从Activity的启动流程说起了，但是这篇文章的目的还是以动态加载为主，Activity以及App的启动流程会有专门的文章去介绍，本文最后给出的参考链接，也会有包含Activity启动流程的介绍。这里挑几个关键链路上的函数简单说下原理：

```java
/**  Core implementation of activity launch. */
private Activity performLaunchActivity(ActivityClientRecord r, Intent customIntent) {
    ActivityInfo aInfo = r.activityInfo;
    
    if (r.packageInfo == null) {
        r.packageInfo = getPackageInfo(aInfo.applicationInfo, r.compatInfo,
                Context.CONTEXT_INCLUDE_CODE);
    }

    // ...
  
    Activity activity = null;
    try {
        
        java.lang.ClassLoader cl = appContext.getClassLoader();
        activity = mInstrumentation.newActivity(
                cl, component.getClassName(), r.intent);
        StrictMode.incrementExpectedActivityCount(activity.getClass());
        r.intent.setExtrasClassLoader(cl);
        r.intent.prepareToEnterProcess();
        if (r.state != null) {
            r.state.setClassLoader(cl);
        }
    } catch (Exception e) {
        if (!mInstrumentation.onException(activity, e)) {
            throw new RuntimeException(
                "Unable to instantiate activity " + component
                + ": " + e.toString(), e);
        }
    }

    // ...
    return activity;
}
```

看注释就知道，这个方法是启动Activity的核心方法，在启动Activity之前会通过调用`getPackageInfo`方法来先获取并解析Activity信息，我们进到这个方法中。



```java
public final LoadedApk getPackageInfo(ApplicationInfo ai, CompatibilityInfo compatInfo,
                                      int flags) {
  // ...
  return getPackageInfo(ai, compatInfo, null, securityViolation, includeCode,
                        registerPackage);
}
```

  这个三个参数的`getPackageInfo`调用了五个参数的`getPackageInfo`方法，注意第三个参数传的是`null`，我们继续进入五个参数的`getPackageInfo`方法。



```java
private LoadedApk getPackageInfo(ApplicationInfo aInfo, CompatibilityInfo compatInfo,
        ClassLoader baseLoader, boolean securityViolation, boolean includeCode,
        boolean registerPackage) {
    final boolean differentUser = (UserHandle.myUserId() != UserHandle.getUserId(aInfo.uid));
    synchronized (mResourcesManager) {
        WeakReference<LoadedApk> ref;
        if (differentUser) {
            // Caching not supported across users
            ref = null;
        } else if (includeCode) {
            ref = mPackages.get(aInfo.packageName);
        } else {
            ref = mResourcePackages.get(aInfo.packageName);
        }

        LoadedApk packageInfo = ref != null ? ref.get() : null;
        if (packageInfo == null || (packageInfo.mResources != null
                && !packageInfo.mResources.getAssets().isUpToDate())) {
            if (localLOGV) Slog.v(TAG, (includeCode ? "Loading code package "
                    : "Loading resource-only package ") + aInfo.packageName
                    + " (in " + (mBoundApplication != null
                            ? mBoundApplication.processName : null)
                    + ")");
            packageInfo =
                new LoadedApk(this, aInfo, compatInfo, baseLoader,
                        securityViolation, includeCode &&
                        (aInfo.flags&ApplicationInfo.FLAG_HAS_CODE) != 0, registerPackage);

            if (mSystemThread && "android".equals(aInfo.packageName)) {
                packageInfo.installSystemApplicationInfo(aInfo,
                        getSystemContext().mPackageInfo.getClassLoader());
            }

            // ...
        }
        return packageInfo;
    }
}
```

有一个HashMap即mPackages维护包名和LoadedApk的对应关系，即每一个应用有一个键值对对应，如果为null，就新创建一个LoadedApk对象，并将其添加到Map中。第一次执行Activity的时候，很显然是没有这个LoadedApk对象的，所以会生成一个新的LoadedApk对象，然后注意到传入了一个baseLoader，正是上面传的`null`。



我们再回头看下`performLaunchActivity`，当调用完`getPackageInfo`之后，会调用`java.lang.ClassLoader cl = appContext.getClassLoader();`去获取`classLoader`，我们进到`ContextImpl.getClassLoader`方法：

```java
@Override
    public ClassLoader getClassLoader() {
        return mClassLoader != null ? mClassLoader : (mPackageInfo != null ? mPackageInfo.getClassLoader() : ClassLoader.getSystemClassLoader());
    }
```

这里的`mClassLoader`正是上面传入的`null`，而`mPackageInfo`是上边生成的`LoadedApk`对象不为空，所以会调用`LoadedApk`的`getClassLoader`方法。这里就不在一层一层的剥开了，因为想节省点笔迹🤪，总之，翻源码到最后，你会发现最终是通过调用`ClassLoader.getSystemClassLoader`来获取一个`classLoader`，而这个`classLoader`正好是`PathClassLoader`。



到这里一切真相大白了吧？前面**我们虽然用`DexClassLoader`通过对APK的动态加载成功加载了`TestActivity`到虚拟机，但是当系统启动该`Activity`的时候，依然会出现加载类失败的异常，因为`Activity`在启动时用到的是`PathClassLoader`**。前面在介绍`Android`的`ClassLoader`的时候提到过，`PathClassLoader`是`Android`默认使用的类加载器，一个`APK`中的`Activity`等类便是在其中加载，但是我们的`TestActivity`不存在于当前的`APK`，而是在外部的`dex`文件上，自然而然的就会出现上边找不到`Activity`的异常了。



那我们是不是可以替换掉这个`PathClassLoader`为`DexClassLoader`不就好了吗？答案是肯定的。除了这个方案之外，我们还可以利用双亲委派的原理，给出另一种方案。两种解决方案如下：

* 替换系统组件类加载器为我们的`DexClassLoader`，同时设置`DexClassLoader`的`parent`为系统组件的类加载器。
* 打破原有的双亲关系，在系统组件类加载器和`BootClassLoader`中插入我们自己的`DexClassLoader`即可。



#### 方案一：替换`mClassLoader`为`DexClassLoader`

![image-20220115213655318](https://img.heshipeng.com/202201152137281.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)修改`MainActivity`的代码如下：

  ```java
  public class MainActivity extends AppCompatActivity {
  
      @Override
      protected void onCreate(Bundle savedInstanceState) {
          super.onCreate(savedInstanceState);
          setContentView(R.layout.activity_main);
          String sdcardPath = Environment.getExternalStorageDirectory().getAbsolutePath();
          startActivityTest(this, sdcardPath + File.separator + "classes3.dex");
      }
  
      private void replaceClassLoader(ClassLoader classLoader) {
          try {
              // 加载ActivityThread类
              Class<?> ActivityThreadClazz = classLoader.loadClass("android.app.ActivityThread");
              // 获取currentActivityThread方法,从而获取ActivityThread实例
              Method currentActivityThreadMethod = ActivityThreadClazz.getDeclaredMethod("currentActivityThread");
              currentActivityThreadMethod.setAccessible(true);
              Object activityThreadObj = currentActivityThreadMethod.invoke(null);
  
              // 获取ActivityThread的mPackage属性
              Field mPackageField = ActivityThreadClazz.getDeclaredField("mPackages");
              mPackageField.setAccessible(true);
              // 获取loadedApk对象
              ArrayMap mPackageObj = (ArrayMap) mPackageField.get(activityThreadObj);
              WeakReference wr = (WeakReference) mPackageObj.get(this.getPackageName());
              Object loadedApkObj = wr.get();
  
              // 替换mClassLoader
              Class loadedApkClazz = classLoader.loadClass("android.app.LoadedApk");
              Field mClassLoader = loadedApkClazz.getDeclaredField("mClassLoader");
              mClassLoader.setAccessible(true);
              mClassLoader.set(loadedApkObj, classLoader);
          } catch (ClassNotFoundException | NoSuchMethodException e) {
              e.printStackTrace();
          } catch (IllegalAccessException e) {
              e.printStackTrace();
          } catch (InvocationTargetException e) {
              e.printStackTrace();
          } catch (NoSuchFieldException e) {
              e.printStackTrace();
          }
      }
  
      public void startActivityTest(Context context, String dexFilePath) {
          Class<?> clazz = null;
          DexClassLoader dexClassLoader = new DexClassLoader(dexFilePath, null, null, MainActivity.class.getClassLoader());
          try {
              replaceClassLoader(dexClassLoader);
              clazz = dexClassLoader.loadClass("com.example.dexclass.TestActivity");
          } catch (ClassNotFoundException e) {
              e.printStackTrace();
          }
          context.startActivity(new Intent(context, clazz));
      }
  }
  ```

运行项目，结果如预期：

![image-20220115213932649](https://img.heshipeng.com/202201152139770.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



#### 方案二：在`mClassLoader`和`BootClassLoader`之间插入`DexClassLoader`

![image-20220115214708638](https://img.heshipeng.com/202201152147745.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

修改`TestActivity`代码如下：

```java
public class TestActivity extends Activity {
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        Log.d("TestActivity", "i am from TestActivity.onCreate");
    }
}
```

把`AppCompatActivity`改为了`Activity`，防止有一些类重复加载。



再次修改`MainActivity`的代码如下：

```java
public class MainActivity extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        String sdcardPath = Environment.getExternalStorageDirectory().getAbsolutePath();
        startActivityTest(this, sdcardPath + File.separator + "classes3.dex");
    }

    public void startActivityTest(Context context, String dexFilePath) {
        Class<?> clazz = null;
        ClassLoader pathClassLoader = MainActivity.class.getClassLoader();
        ClassLoader bootClassLoader = MainActivity.class.getClassLoader().getParent();
        // dexClassLoader的parent为BootClassLoader
        DexClassLoader dexClassLoader = new DexClassLoader(dexFilePath, null, null, bootClassLoader);

        try {
            Field parentField = ClassLoader.class.getDeclaredField("parent");
            parentField.setAccessible(true);
            // 当前组件的ClassLoader的parent为DexClassLoader
            parentField.set(pathClassLoader, dexClassLoader);
            clazz = dexClassLoader.loadClass("com.example.dexclass.TestActivity");
        } catch (ClassNotFoundException | NoSuchFieldException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }
        context.startActivity(new Intent(context, clazz));
    }
}
```

运行项目，结果也如预期：

![image-20220115231522507](https://img.heshipeng.com/202201152315708.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



### 总结

动态加载就是用到的时候再去加载，也叫懒加载，也就意味着用不到的时候是不会去加载的。动态加载是dex加壳，插件化，热更新的基础。动态加载的dex不具有生命周期特征，App中的Activity, Service等组件无法正常工作，只能完成一般函数的调用；需要对ClassLoader进行修正，App才能正常运行，两种修正方案：

1. 替换系统组件类加载器为我们的`DexClassLoader`，同时设置`DexClassLoader`的`parent`为系统组件的类加载器。
2. 打破原有的双亲关系，在系统组件类加载器和`BootClassLoader`中插入我们自己的`DexClassLoader`即可。



### 参考链接

[Android动态加载Activity原理](https://blog.csdn.net/cauchyweierstrass/article/details/51087198)

[FART：ART环境下基于主动调用的自动化脱壳方案](https://bbs.pediy.com/thread-252630.htm#msg_header_h2_6)

[ActivityThread源码](http://androidxref.com/9.0.0_r3/xref/frameworks/base/core/java/android/app/ActivityThread.java)

[Activity的启动流程探究](https://samiu.top/2020/04/16/Activity%E7%9A%84%E5%90%AF%E5%8A%A8%E6%B5%81%E7%A8%8B%E6%8E%A2%E7%A9%B6/)

[Android动态加载基础 ClassLoader工作机制](https://segmentfault.com/a/1190000004062880)
