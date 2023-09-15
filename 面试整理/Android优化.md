Android 优化相关

[toc]



# 布局优化

检测工具，

1. 开发者工具中的检测过度绘制
2. Hierarchy Viewer
3. Systrace

优化的方向：

1. 减少嵌套层级、去除没必要的背景，避免过度绘制
2. 自定义控件View优化：使用 clipRect() 、 quickReject()，减少全局绘制
3. 使用ViewStub、merge 标签
4. 硬件加速



# 内存优化

## **常见的内存泄漏**

|                           | 导致原因                             | 解决办法                                  |
| ------------------------- | ------------------------------------ | ----------------------------------------- |
| Handler                   | 延时任务持有Activity等销毁对象的引用 | 销毁时Remove掉、静态Handler               |
| 单例                      | 传入了Activity等的一些引用           | 使用Application Context                   |
| WebView                   | Context 引用泄漏                     | 使用Application Context，并且及时销毁回收 |
| Bitmap                    | 持有Activity等销毁对象的引用         | 使用后recycle                             |
| 动画                      | 界面销毁，动画未取消                 | 销毁时停止动画                            |
| 流                        | 使用后未及时关闭                     | 使用完及时关闭                            |
| ContentProvider、数据库   | Cursor 未Close                       | 及时关闭Cursor                            |
| EditText的TextWatcher等   | Edittext引用                         | 及时remove                                |
| Activity.getSystemService | Context 引用泄漏                     | 使用Application Context                   |

在Activity onDestory的时候，遍历View树，清空backGround、Drawable、EditText的TextWatcher等

## 检测内存泄漏

1. MAT

   [官方文档](http://wiki.eclipse.org/MemoryAnalyzer)

2. Android Studio的 profiler

   [官网文档](https://developer.android.com/studio/profile/memory-profiler)

3. LeakCanary

前两个都是可视化的，当时一般的时候，我们用 LeakCanary 来检测内存泄漏，原理是

1. 初始化：直接debugImplementation就能实现，他是在 ContentProvider里面做的初始化，当打包的时候，会合并各个清单文件。里面注册的 ContentProvider，ContentProvider 会在 Application的 attachBaseContext 之后， onCreate之前创建。
2. 引用队列可以配合软引用、弱引用及虚引用使用，引用的对象将要被JVM回收时，会将其加入到引用队列中。
3. 注册 Application.ActivityLifecycleCallbacks 监听Activity的生命周期，以及 fragmentManager.registerFragmentLifecycleCallbacks监听Fragment的生命周期。
4. 比如监听 onActivityDestroyed方法，当监听到这个方法调用的时候，把Activity 全部放到观察数组中，并且用引用队列包裹这个activity，生成key（UUID），然后过5s，看看引用队列里面有没有这个key，如果有，证明回收了，然后把观察数组中的remove掉这个key，此时如果这个数组里面的count > 0 ，证明有可能是怀疑的泄漏，然后 调用 Runtime.getRuntime().gc()，之后再 看看 引用队列有没有这个数据，如果有，然后把观察数组中的remove掉这个key，之后再看观察数组中的count，如果小于5，只是提示一下。如果count 大于 5 （防止卡顿），就开始使用 shark （2.0之前是haha）分析堆栈信息。用可达到性分析，找到最短的链路，

## 优化的方向和策略

| 方向     | 策略                                                         |
| -------- | ------------------------------------------------------------ |
| 引用类型 | 适当位置使用软引用等代替强引用，如图片引用                   |
| Bitmap   | RGB_565代替RGB_8888<br />使用inSampleSize避免不必要的大图加载<br />使用后及时释放。 |
| 内存抖动 | 避免在For等循环体中分配多个临时变量<br />减少在onDraw中初始化对象 |
| 线程     | 避免直接new Thread，尽量使用线程池                           |
| 监听回调 | 在 Activity.onTrimMemory() Fragement.OnTrimMemory() Service.onTrimMemory() ContentProvider.OnTrimMemory() 类中实现 ComponentCallbacks2 接口，或者是在 Application 重写onTrimMemory方法，监听低内存的回调，做相应的释放回调[官方文档](https://developer.android.com/topic/performance/memory) |
| Service  | 不再使用时应该销毁它，建议使用IntentServcie                  |

# 启动优化

[参考链接](https://juejin.im/post/6844903958113157128#heading-8)

## apk编译流程

**Android Studio 按下编译按钮后发生了什么？**

1. 打包资源文件，生成R.java文件（使用工具AAPT）
2. 处理AIDL文件，生成java代码（没有AIDL则忽略）
3. 编译 java 文件，生成对应.class文件（java compiler）
4. .class 文件转换成dex文件（dex）
5. 打包成没有签名的apk（使用工具apkbuilder）
6. 使用签名工具给apk签名（使用工具Jarsigner）
7. 对签名后的.apk文件进行对齐处理，不进行对齐处理不能发布到Google Market（使用工具zipalign）

在第4步，将class文件转换成dex文件，默认只会生成一个dex文件，单个dex文件中的方法数不能超过65536，不然编译会报错：

> Unable to execute dex: method ID not in [0, 0xffff]: 65536

App集成一堆库之后，方法数一般都是超过65536的，解决办法就是：一个dex装不下，用多个dex来装，gradle增加一行配置即可。

> multiDexEnabled true

这样解决了编译问题，在5.0以上手机运行正常，但是5.0以下手机运行直接crash，报错 Class NotFound xxx。

Android 5.0以下，ClassLoader加载类的时候只会从class.dex（主dex）里加载，ClassLoader不认识其它的class2.dex、class3.dex、...，当访问到不在主dex中的类的时候，就会报错:Class NotFound xxx，因此谷歌给出兼容方案，`MultiDex`。

## MultiDex

用来解决android 5.0之前dalvik 虚拟机出现的65536问题，

继承MultiDexApplication，如果项目中已经自定义了application并且继承了其他必须继承的application 则需要重写 applicaiton的 attachBaseContext方法并初始化MultiDex。

### MultiDex 局限性

1. 应用程序首次启动时 Dalvik虚拟机 会对所有的dex执行一个dexopt操作，生成odex文件，这个过程很复杂也很耗时，如果应用的从dex文件太大，可能会导致出现anr。

2. 引入MultiDex机制后，必然出现主dex和从dex，应用启动所需要的类都必须放到主dex，否则会出现 NoClassDefFoundError 错误，如果应用中我们自己应用的第三方库 有通过反射java类，或者调用NDK层代码的java方法，这些可能就不会放到主dex文件中，如果在应用启动时需要用到，必然出现问题。

3. 2.dalvik 的线性内存分配器 linearAlloc 本身有一个bug，使用MultiDex分包的应用会启动失败。

   所以建议将应用的minSdkVersion 指定为14来规避可能发生的问题。

### MultiDex优化

#### 方案1：子线程install（不推荐）

这个方法大家很容易就能想到，在闪屏页开一个子线程去执行`MultiDex.install`，然后加载完才跳转到主页。需要注意的是闪屏页的Activity，包括闪屏页中引用到的其它类必须在主dex中，不然在`MultiDex.install`之前加载这些不在主dex中的类会报错Class Not Found。这个可以通过gradle配置

缺点：

1. MultiDex加载逻辑放在闪屏页的话，闪屏页中引用到的类都要配置在主dex。
2. ContentProvider必须在主dex，一些第三方库自带ContentProvider，维护比较麻烦，要一个一个配置。

#### 方案2：今日头条方案

逻辑如下：
 **1. 创建临时文件，作为判断MultiDex是否加载完的条件**
 **2. 启动LoadDexActivity去加载MultiDex（LoadDexActivity在单独进程），加载完会删除临时文件**
 **3. 开启while循环，直到临时文件不存在才跳出循环，进入Application的onCreate**

今日头条在5.0以下手机首次启动应该是这样：

1. 打开桌面图标
2. 显示默认背景
3. 跳转到加载dex的界面，展示一个loading的加载框几秒钟
4. 跳转到闪屏页

方案总结：在主进程Application 的 attachBaseContext 方法中判断如果需要使用MultiDex，则创建一个临时文件，然后开一个进程（LoadDexActivity），显示Loading，异步执行MultiDex.install 逻辑，执行完就删除临时文件并finish自己。

主进程Application 的 attachBaseContext 进入while代码块，定时轮循临时文件是否被删除，如果被删除，说明MultiDex已经执行完，则跳出循环，继续正常的应用启动流程。

注意LoadDexActivity 必须要配置在main dex中。

## 优化的方向和策略

| 方向                       | 策略                                                         |
| -------------------------- | ------------------------------------------------------------ |
| 闪屏页优化                 | mainefast 中Style，障眼法，背景与启动页相同                  |
| 预创建Activity（今日头条） | 对象第一次创建的时候，java虚拟机首先检查类对应的Class 对象是否已经加载。如果没有加载，jvm会根据类名查找.class文件，将其Class对象载入。同一个类第二次new的时候就不需要加载类对象，而是直接实例化，创建时间就缩短了。 |
| 第三方库懒加载             | 很多第三方开源库都说在Application中进行初始化，十几个开源库都放在Application中，肯定对冷启动会有影响，所以可以考虑按需初始化，例如Glide，可以放在自己封装的图片加载类中，调用到再初始化，其它库也是同理，让Application变得更轻。 |
| WebView启动优化            | WebView第一次创建比较耗时，可以预先创建WebView，提前将其内核初始化。 <br />使用WebView缓存池，用到WebView的地方都从缓存池取，缓存池中没有缓存再创建，注意内存泄漏问题。<br /> 本地预置html和css，WebView创建的时候先预加载本地html，之后通过js脚本填充内容部分。 |
| 数据预加载                 | 这种方式一般是在主页空闲的时候，将其它页面的数据加载好，保存到内存或数据库，等到打开该页面的时候，判断已经预加载过，直接从内存或数据库读取数据并显示。 |

# APK瘦身

APK 组成

| 名称        | 说明                                                         |
| ----------- | ------------------------------------------------------------ |
| 代码相关    | classes.dex文件                                              |
| 资源相关    | res、assets、编译后的二进制资源文件 resources.arsc 和 清单文件 等等 |
| so          | lib目录下文件                                                |
| MANIFEST.MF | 其中每一个资源文件都有一个对应的 SHA-256-Digest（SHA1) 签名，MANIFEST.MF 文件的 SHA256（SHA1） 经过 base64 编码的结果即为 CERT.SF 中的 SHA256（SHA1）-Digest-Manifest 值。 |
| CERT.SF     | 除了开头处定义的 SHA256（SHA1）-Digest-Manifest 值，后面几项的值是对 MANIFEST.MF 文件中的每项再次 SHA256（SHA1） 经过 base64 编码后的值。 |
| CERT.RSA    | 其中包含了公钥、加密算法等信息。首先，对前一步生成的MANIFEST.MF使用了SHA256（SHA1）-RSA算法，用开发者私钥签名。然后，在安装时使用公钥解密。最后，将其与未加密的摘要信息（MANIFEST.MF文件）进行对比，如果相符，则表明内容没有被修改。 |



## 优化方向与策略

1. 移除无用代码、无用资源（lint）
2. 压缩图片大小（TinyPng）、png替换为webp/jpeg(透明图片需要png)
3. 语言对应资源文件
4. 引用第三方时，只引入需要部分，
5. so文件移除， **arm64-v8** 向下兼容 **armeabi-v7a** 向下兼容 **armeabi**，**armeabi**被称为万金油，目前微信支付宝为 **armeabi-v7a**

## 优化的方向和策略

| 方向     | 策略                                                         |
| -------- | ------------------------------------------------------------ |
| 资源     | 移除无用代码、无用资源（lint）、语言对应资源文件             |
| 图片     | 压缩图片大小（TinyPng）<br />png替换为webp/jpeg(透明图片需要png) |
| 第三方库 | 只引入需要部分，                                             |
| so       | so文件移除， **arm64-v8** 向下兼容 **armeabi-v7a** 向下兼容 **armeabi**，**armeabi**被称为万金油，目前微信支付宝为 **armeabi-v7a** |

# 