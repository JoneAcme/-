**Android 应用启动相关**



[TOC]





# Android 系统的启动

转自：[原文链接](https://juejin.im/post/6884785484766117895)

`init进程`是Linux内核启动完成后在用户空间启动的**第一个进程**，主要负责**初始化工作**、**启动属性服务**、**解析init.rc文件并启动Zygote进程**。

`Zygote进程`是一个**进程孵化器**，负责**创建虚拟机实例**、**应用程序进程**、**系统服务进程SystemServer**。他通过fork（复制进程）的方式创建子进程，子进程能**继承**父进程的系统资源如常用类、注册的JNI函数、主题资源、共享库等。

由于Zygote进程启动时会创建虚拟机实例，由Zygote fork出的应用程序进程和SystemServer进程则可以在内部获取到一个虚拟机实例副本。

## Zygote启动

init进程会解析配置文件[init.rc](https://www.androidos.net.cn/android/8.0.0_r4/xref//system/core/rootdir/init.rc)，来启动一些需要在开机时就启动的系统进程，如Zygote进程、ServiceManager进程等。init.rc是由Android初始化语言编写的脚本配置。

**Zygote进程是使用socket来进行跨进程通信的**

init进程启动后，通过**fork和execve**来启动Zygote进程

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ec10262d6b974dc4bf7aee5bb5a8ccbe~tplv-k3u1fbpfcp-zoom-1.image)

**小结：**

**init进程读取配置文件init.rc后，fork出Zygote进程，通过execve函数执行Zygote的执行程序app_process，进入ZygoteInit类的main函数**。



**native层app_main**

app_main.cpp的main函数会调用runtime.start()

```java
//frameworks/base/core/jni/AndroidRuntime.cpp
void AndroidRuntime::start(...){
    //1. 启动java虚拟机
    if (startVm(&mJavaVM, &env, zygote) != 0) {
        return;
    }
    //2. 为java虚拟机注册JNI方法
    if (startReg(env) < 0) {
        return;
    }
    //根据传入的参数找到ZygoteInit类和他的main函数
    //3. 通过JNI调用ZygoteInit的main函数
    env->CallStaticVoidMethod(startClass, startMeth, strArray);
}
```

**Java层ZygoteInit**

[ZygoteInit](https://www.androidos.net.cn/android/8.0.0_r4/xref//frameworks/base/core/java/com/android/internal/os/ZygoteInit.java)的main函数

```java
//ZygoteInit.java
public static void main(String argv[]) {
    //是否要创建SystemServer
    boolean startSystemServer = false;
    //默认的socket名字
    String socketName = "zygote";
    //是否要延迟资源的预加载
    boolean enableLazyPreload = false;

    for (int i = 1; i < argv.length; i++) {
        if ("start-system-server".equals(argv[i])) {
            //在init.rc文件中，有--start-system-server参数，表示要创建SystemServer
            startSystemServer = true;
        } else if ("--enable-lazy-preload".equals(argv[i])) {
            //init.rc没有这个参数，资源的预加载不会被延迟
            enableLazyPreload = true;
        } else if (argv[i].startsWith(SOCKET_NAME_ARG)) {
            //init.rc可以通过--socket-name=指定socket名字来覆盖默认值
            socketName = argv[i].substring(SOCKET_NAME_ARG.length());
        }
    }

    //1. 创建服务端socket，名字为socketName即zygote
    zygoteServer.registerServerSocket(socketName);

    if (!enableLazyPreload) {
        //2. 没有被延迟，就预加载资源
        preload(bootTimingsTraceLog);
    }

    if (startSystemServer) {
        //3. fork并启动SystemServer进程
        startSystemServer(abiList, socketName, zygoteServer);
    }

    //4. 等待AMS请求（AMS会通过socket请求Zygote来创建应用程序进程）
    zygoteServer.runSelectLoop(abiList);
}
```

native层的3个环节和Java层的4个环节：

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7a57768205744cec9c540ace6947c46b~tplv-k3u1fbpfcp-zoom-1.image)



## SystemServer启动

`SystemServer`进程主要负责**创建启动系统服务如AMS、WMS和PMS等**。

SystemServer进程由Zygote进程fork出来并启动，在ZygoteInit类中，

```java
//ZygoteInit.java
private static boolean startSystemServer(...){
    String args[] = {
        //...
        //启动的类名：
        "com.android.server.SystemServer",
    };
    //fork进程，由native层实现
    pid = Zygote.forkSystemServer();
    //处理SystemServer进程
    handleSystemServerProcess(parsedArgs);
}

private static void handleSystemServerProcess(...){
    ZygoteInit.zygoteInit(...);
}

public static final void zygoteInit(...){
    //启动binder线程池
    ZygoteInit.nativeZygoteInit();
    //内部经过层层调用，找到"com.android.server.SystemServer"类和他的main函数，然后执行
    RuntimeInit.applicationInit(targetSdkVersion, argv, classLoader);
}
```

这里启动了binder线程池，**SystemServer进程就可以用binder机制来跨进程通信了**（Zygote进程是用socket来通信的），接着进入了[SystemServer](https://www.androidos.net.cn/android/8.0.0_r4/xref//frameworks/base/services/java/com/android/server/SystemServer.java)的main函数，

```java
//SystemServer.java
public static void main(String[] args) {
    new SystemServer().run();
}

private void run() {
    //创建looper
    Looper.prepareMainLooper();
    //加载动态库libandroid_servers.so
    System.loadLibrary("android_servers");
    //创建系统上下文
    createSystemContext();

    //创建SSM，用于服务的创建、启动和生命周期管理
    mSystemServiceManager = new SystemServiceManager(mSystemContext);
    
    //服务根据优先级被分成3批来启动：
    //启动引导服务，如AMS、PMS等
    startBootstrapServices();
    //启动核心服务
    startCoreServices();
    //启动其他服务
    startOtherServices();

    //开启looper循环
    Looper.loop();
}
```

AMS的启动：

```java
//SystemServer.java
private void startBootstrapServices() {
    //由SSM创建启动
    mActivityManagerService = mSystemServiceManager.startService(
        ActivityManagerService.Lifecycle.class).getService();
    mActivityManagerService.setSystemServiceManager(mSystemServiceManager);
}

private void startOtherServices() {
    //AMS准备就绪
    mActivityManagerService.systemReady(...);
}
```

**SystemServer进程被创建后，主要做了3件事情：**

1. **启动binder线程池**
2. **创建SystemServiceManager（SSM）**
3. **用SSM启动各种服务**。

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/45bd8fbfeb0c43d6a16286d7ce6688c6~tplv-k3u1fbpfcp-zoom-1.image)

## Launcher的启动

`Launcher`作为Android的桌面，用于**管理应用图标和桌面组件**。

前边可知SystemServer进程会启动各种服务，其中**PackageManagerService启动后会将系统中的应用程序安装完成**，然后**由AMS来启动Launcher**。

```java
//SystemServer.java
private void startOtherServices() {
    //AMS准备就绪
    mActivityManagerService.systemReady(...);
}
复制代码
```

跟进[ActivityManagerService](https://www.androidos.net.cn/android/8.0.0_r4/xref/frameworks/base/services/core/java/com/android/server/am/ActivityManagerService.java)，

```java
//ActivityManagerService.java
public void systemReady(...) {
    //经过层层调用来到startHomeActivityLocked
}

boolean startHomeActivityLocked(...) {
    //最终会启动Launcher应用的Activity
    mActivityStarter.startHomeActivityLocked(...);
}
复制代码
```

Activity类是[Launcher.java](https://www.androidos.net.cn/android/8.0.0_r4/xref/packages/apps/Launcher3/src/com/android/launcher3/Launcher.java)，剩下的流程就是加载已安装的应用程序信息，然后展示，就不具体分析了。

![img](https:////p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9d60c16289b34266a8e29af79b78c348~tplv-k3u1fbpfcp-zoom-1.image)

## 总结

Android系统启动的核心流程如下：

1. Linux内核启动
2. init进程启动
3. init进程fork出Zygote进程
4. Zygote进程fork出SystemServer进程
5. SystemServer进程启动各项服务（PMS、AMS等）
6. AMS服务启动Launcher桌面

![img](https:////p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8cbc79f8743847a98af2cb49aedafcc0~tplv-k3u1fbpfcp-zoom-1.image)

Zygote进程启动好服务端socket后，便会等待AMS的socket请求，来创建应用程序进程。

## 细节补充

1. **Zygote的跨进程通信没有使用binder，而是socket，所以应用程序进程的binder机制不是继承而来，而是进程创建后自己启动的。**

2. **Zygote跨进程通信之所以用socket而不是binder，是因为binder通信是多线程的（多线程程序里不准使用fork），而Zygote需要在单线程状态下fork子进程来避免死锁问题。**[参考链接](https://blog.csdn.net/qq_39037047/article/details/88066589)

3. **PMS、AMS等系统服务启动后会调用ServiceManager.addService()注册，然后运行在自己的工作线程。**

# Android应用进程的启动

转自：[原文链接](https://juejin.im/post/6887431834041483271)

系统启动完成后，由Zygote进程fork出的SystemServer进程会启动各项系统服务，其中就包含了AMS，AMS会启动Launcher桌面，此时就可以等待用户点击App图标来启动应用进程了。

系统服务的启动，不管是**由init进程启动的独立进程的系统服务**如SurfaceFlinger，还是**由SystemServer进程启动的非独立进程的系统服务**如AMS，都是在**ServiceManager进程**中完成注册和获取的，在跨进程通信上使用了Android的binder机制。

![img](https:////p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8f27d1c3f4a44815bdd9a1168c7221cd~tplv-k3u1fbpfcp-zoom-1.image)

**ServiceManager进程**本身也是一个系统服务，经过**启动进程**、**启动binder机制**、**发布自己**和**等待请求**4个步骤，就可以处理其他系统服务的获取和注册需求了。

## AMS发送socket请求

Android应用进程的启动是**被动式**的，在Launcher桌面点击图标启动一个应用的组件如Activity时，如果Activity所在的进程不存在，就会创建并启动进程。

点击App图标后经过层层调用会来到[ActivityStackSupervisor](https://www.androidos.net.cn/android/8.0.0_r4/xref/frameworks/base/services/core/java/com/android/server/am/ActivityStackSupervisor.java)的startSpecificActivityLocked方法，

```java
//ActivityStackSupervisor.java
final ActivityManagerService mService;

void startSpecificActivityLocked(...) {
    //查找Activity所在的进程，ProcessRecord是用来封装进程信息的数据结构
    ProcessRecord app = mService.getProcessRecordLocked(...);
	//如果进程已启动，并且binder句柄IApplicationThread也拿到了，那就直接启动Activity
    if (app != null && app.thread != null) {
        realStartActivityLocked(r, app, andResume, checkConfig);
        return;
    }
	//否则，让AMS启动进程
    mService.startProcessLocked(...);
}
复制代码
```

**app.thread并不是线程，而是一个binder句柄**。应用进程使用AMS需要拿到AMS的句柄IActivityManager，而系统需要通知应用和管理应用的生命周期，所以也需要持有应用进程的binder句柄IApplicationThread。

他们**互相持有彼此的binder句柄，来实现双向通信**。

![img](https:////p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e2eb7257f36e443f91dd857683be11b6~tplv-k3u1fbpfcp-zoom-1.image)

那IApplicationThread句柄是怎么传给AMS的呢？

Zygote进程收到socket请求后会处理请求参数，执行[ActivityThread](https://www.androidos.net.cn/android/8.0.0_r4/xref/frameworks/base/core/java/android/app/ActivityThread.java)的入口函数main，

```java
//ActivityThread.java
public static void main(String[] args) {
    //创建主线程的looper
    Looper.prepareMainLooper();
    //ActivityThread并不是线程，只是普通的java对象
    ActivityThread thread = new ActivityThread();
    //告诉AMS，应用已经启动好了
    thread.attach(false);
	//运行looper，启动消息循环
    Looper.loop();
}

private void attach(boolean system) {
    //获取AMS的binder句柄IActivityManager
    final IActivityManager mgr = ActivityManager.getService();
    //告诉AMS应用进程已经启动，并传入应用进程自己的binder句柄IApplicationThread
    mgr.attachApplication(mAppThread);
}
复制代码
```

所以对于AMS来说，

1. **AMS向Zygote发起启动应用的socket请求，Zygote收到请求fork出进程，返回进程的pid给AMS；**
2. **应用进程启动好后，执行入口main函数，通过attachApplication方法告诉AMS已经启动，同时传入应用进程的binder句柄IApplicationThread。**

完成这两步，应用进程的启动过程才算完成。

AMS的startProcessLocked启动应用进程时都做了些什么。

```java
//ActivityManagerService.java
final ProcessRecord startProcessLocked(...){
    ProcessRecord app = getProcessRecordLocked(processName, info.uid, keepIfLarge);
    //如果进程信息不为空，并且已经拿到了Zygote进程返回的应用进程pid
    //说明AMS已经请求过了，并且Zygote已经响应请求然后fork出进程了
    if (app != null && app.pid > 0) {
        //但是app.thread还是空，说明应用进程还没来得及注册自己的binder句柄给AMS
        //即此时进程正在启动，那就直接返回，避免重复创建
        if (app.thread == null) {
            return app;
        }
    }
    //调用重载方法
    startProcessLocked(...);
}
复制代码
```

之所以要判断app.thread，是为了避免当应用进程正在启动的时候，假如又有另一个组件需要启动，导致重复拉起（创建）应用进程。

继续看重载方法startProcessLocked，

```java
//ActivityManagerService.java
private final void startProcessLocked(...){
    //应用进程的主线程的类名
    if (entryPoint == null) entryPoint = "android.app.ActivityThread";
    ProcessStartResult startResult = Process.start(entryPoint, ...);
}

//Process.java
public static final ProcessStartResult start(...){
    return zygoteProcess.start(...);
}
复制代码
```

来到ZygoteProcess，

```java
//ZygoteProcess.java
public final Process.ProcessStartResult start(...){
    return startViaZygote(...);
}

private Process.ProcessStartResult startViaZygote(...){
    ArrayList<String> argsForZygote = new ArrayList<String>();
    //...处理各种参数
    return zygoteSendArgsAndGetResult(openZygoteSocketIfNeeded(abi), argsForZygote);
}
复制代码
```

其中：

1. openZygoteSocketIfNeeded打开本地socket
2. zygoteSendArgsAndGetResult发送请求参数，其中带上了ActivityThread类名
3. return返回的数据结构ProcessStartResult中会有pid字段

梳理一下：

![img](https:////p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5b4acc1641604d26903ccf53755e2d84~tplv-k3u1fbpfcp-zoom-1.image)

> 注意：Zygote进程启动时已经创建好了虚拟机实例，所以由他fork出的应用进程可以直接继承过来用而无需创建。

下面来看Zygote是如何处理socket请求的。

## Zygote处理socket请求

从 [图解Android系统的启动](https://mp.weixin.qq.com/s/xsoc9omtu-gGAFG1pCsCsQ) 一文可知，在ZygoteInit的main函数中，会创建服务端socket，

```java
//ZygoteInit.java
public static void main(String argv[]) {
    //Server类，封装了socket
    ZygoteServer zygoteServer = new ZygoteServer();
    //创建服务端socket，名字为socketName即zygote
    zygoteServer.registerServerSocket(socketName);
    //进入死循环，等待AMS发请求过来
    zygoteServer.runSelectLoop(abiList);
}
复制代码
```

看到ZygoteServer，

```java
//ZygoteServer.java
void registerServerSocket(String socketName) {
    int fileDesc;
    //socket真正的名字被加了个前缀，即 "ANDROID_SOCKET_" + "zygote"
    final String fullSocketName = ANDROID_SOCKET_PREFIX + socketName;

    String env = System.getenv(fullSocketName);
    fileDesc = Integer.parseInt(env);

    //创建文件描述符fd
    FileDescriptor fd = new FileDescriptor();
    fd.setInt$(fileDesc);
    //创建LocalServerSocket对象
    mServerSocket = new LocalServerSocket(fd);
}

void runSelectLoop(String abiList){
    //进入死循环
    while (true) {
        for (int i = pollFds.length - 1; i >= 0; --i) {
            if (i == 0) {
                //...
            } else {
                //得到一个连接对象ZygoteConnection，调用他的runOnce
                boolean done = peers.get(i).runOnce(this);
            }
        }
    }
}
复制代码
```

来到ZygoteConnection的runOnce，

```java
//ZygoteConnection.java
boolean runOnce(ZygoteServer zygoteServer){
    //读取socket请求的参数列表
    String args[] = readArgumentList();
    //创建应用进程
    int pid = Zygote.forkAndSpecialize(...);
    if (pid == 0) {
        //如果是应用进程(Zygote fork出来的子进程)，处理请求参数
        handleChildProc(parsedArgs, descriptors, childPipeFd, newStderr);
        return true;
    } else {
        return handleParentProc(pid, descriptors, serverPipeFd, parsedArgs);
    }
}
复制代码
```

handleChildProc方法调用了ZygoteInit的zygoteInit方法，里边主要做了3件事：

1. 启动binder线程池（后面分析）
2. 读取请求参数拿到ActivityThread类并执行他的main函数，执行thread.attach告知AMS并回传自己的binder句柄
3. 执行Looper.loop()启动消息循环（代码前面有）

这样应用进程就启动起来了。梳理一下，

![img](https:////p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7e4e661c1a304b55bacdb566273028ae~tplv-k3u1fbpfcp-zoom-1.image)

下面看下binder线程池是怎么启动的。

## 启动binder线程池

Zygote的跨进程通信没有使用binder，而是socket，所以应用进程的binder机制不是继承而来，而是进程创建后自己启动的。

前边可知，Zygote收到socket请求后会得到一个ZygoteConnection，他的runOnce会调用handleChildProc，

```java
//ZygoteConnection.java
private void handleChildProc(...){
    ZygoteInit.zygoteInit(...);
}

//ZygoteInit.java
public static final void zygoteInit(...){
    RuntimeInit.commonInit();
    //进入native层
    ZygoteInit.nativeZygoteInit();
    RuntimeInit.applicationInit(targetSdkVersion, argv, classLoader);
}
复制代码
```

来到[AndroidRuntime.cpp](https://www.androidos.net.cn/android/8.0.0_r4/xref/frameworks/base/core/jni/AndroidRuntime.cpp)，

```c
//AndroidRuntime.cpp
static void com_android_internal_os_ZygoteInit_nativeZygoteInit(JNIEnv* env, jobject clazz){
    gCurRuntime->onZygoteInit();
}
复制代码
```

来到[app_main.cpp](https://www.androidos.net.cn/android/8.0.0_r4/xref/frameworks/base/cmds/app_process/app_main.cpp)，

```c
//app_main.cpp
virtual void onZygoteInit()
{
    //获取单例
    sp<ProcessState> proc = ProcessState::self();
    //在这里启动了binder线程池
    proc->startThreadPool();
}
复制代码
```

看下[ProcessState.cpp](https://www.androidos.net.cn/android/8.0.0_r4/xref/frameworks/native/libs/binder/ProcessState.cpp)，

```c
//ProcessState.cpp
sp<ProcessState> ProcessState::self()
{
    //单例模式，返回ProcessState对象
    if (gProcess != NULL) {
        return gProcess;
    }
    gProcess = new ProcessState("/dev/binder");
    return gProcess;
}

//ProcessState构造函数
ProcessState::ProcessState(const char *driver)
    : mDriverName(String8(driver))
        , mDriverFD(open_driver(driver)) //打开binder驱动
        ,//...
{
    if (mDriverFD >= 0) {
        //mmap是一种内存映射文件的方法，把mDriverFD映射到当前的内存空间
        mVMStart = mmap(0, BINDER_VM_SIZE, PROT_READ, 
                        MAP_PRIVATE | MAP_NORESERVE, mDriverFD, 0);
    }
}

//启动了binder线程池
void ProcessState::startThreadPool()
{
    if (!mThreadPoolStarted) {
        mThreadPoolStarted = true;
        spawnPooledThread(true);
    }
}

void ProcessState::spawnPooledThread(bool isMain)
{
    if (mThreadPoolStarted) {
        //创建线程名字"Binder:${pid}_${自增数字}"
        String8 name = makeBinderThreadName();
        sp<Thread> t = new PoolThread(isMain);
        //运行binder线程
        t->run(name.string());
    }
}
复制代码
```

ProcessState有两个宏定义值得注意一下，感兴趣可以看 [一次Binder通信最大可以传输多大的数据](https://www.jianshu.com/p/ea4fc6aefaa8) 这篇文章，

```c
//ProcessState.cpp
//一次Binder通信最大可以传输的大小是 1MB-4KB*2
#define BINDER_VM_SIZE ((1 * 1024 * 1024) - sysconf(_SC_PAGE_SIZE) * 2)
//binder驱动的文件描述符fd被限制了最大线程数15
#define DEFAULT_MAX_BINDER_THREADS 15
复制代码
```

我们看下binder线程PoolThread长啥样，

```c
class PoolThread : public Thread
{
public:
    explicit PoolThread(bool isMain)
        : mIsMain(isMain){}
protected:
    virtual bool threadLoop()
    {	//把binder线程注册进binder驱动程序的线程池中
        IPCThreadState::self()->joinThreadPool(mIsMain);
        return false;
    }
    
    const bool mIsMain;
};
复制代码
```

来到[IPCThreadState.cpp](https://www.androidos.net.cn/android/8.0.0_r4/xref/frameworks/native/libs/binder/IPCThreadState.cpp)，

```c
//IPCThreadState.cpp
void IPCThreadState::joinThreadPool(bool isMain)
{
    //向binder驱动写数据：进入死循环
    mOut.writeInt32(isMain ? BC_ENTER_LOOPER : BC_REGISTER_LOOPER);
    status_t result;
    do {
        //进入死循环，等待指令的到来
        result = getAndExecuteCommand();
    } while (result != -ECONNREFUSED && result != -EBADF);
    //向binder驱动写数据：退出死循环
    mOut.writeInt32(BC_EXIT_LOOPER);
}

status_t IPCThreadState::getAndExecuteCommand()
{
    //从binder驱动读数据，得到指令
    cmd = mIn.readInt32();
    //执行指令
    result = executeCommand(cmd);
    return result;
}
复制代码
```

梳理一下binder的启动过程：

1. 打开binder驱动
2. 映射内存，分配缓冲区
3. 运行binder线程，进入死循环，等待指令

## 总结

综上，Android应用进程的启动可以总结成以下步骤：

1. 点击Launcher桌面的App图标
2. AMS发起socket请求
3. Zygote进程接收请求并处理参数
4. Zygote进程fork出应用进程，应用进程继承得到虚拟机实例
5. 应用进程启动binder线程池、运行ActivityThread类的main函数、启动Looper循环

完整流程图：

![img](https:////p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6631644fc5c7402cb5363a5a65e8f009~tplv-k3u1fbpfcp-zoom-1.image)



## 细节补充

- 抛异常清空堆栈帧：Zygote不是直接执行ActivityThread的main函数的，而是通过抛出一个异常进行捕获，捕获后再执行，这样可以清除初始化过程产生的调用堆栈，让ActivityThread的main函数看起来像个应用程序进程的入口函数。

# Application启动

## 涉及到的主要类说明

- **Instrumentation**

Instrumentation会在应用程序的任何代码运行之前被实例化,它能够允许你监视应用程序和系统的所有交互.它还会构造Application,构建Activity,以及生命周期都会经过这个对象去执行.

- **ActivityManagerService**

Android核心服务,简称AMS,负责调度各应用进程,管理四大组件.实现了IActivityManager接口,应用进程能通过Binder机制调用系统服务.

- **LaunchActivityItem**

相当于是一个消息对象,当ActivityThread接收到这个消息则去启动Activity.收到消息后执行execute方法启动activity.

- **ActivityThread**

应用的主线程.程序的入口.在main方法中开启loop循环,不断地接收消息,处理任务.

**Android应用进程的启动**后，会运行ActivityThread类的main函数：

```java
public static void main(String[] args){
    ...
    Looper.prepareMainLooper(); 
    //初始化Looper
    ...
    ActivityThread thread = new ActivityThread();
    //实例化一个ActivityThread
    thread.attach(false);
    //这个方法最后就是为了发送出创建Application的消息
    ... 
    Looper.loop();
    //主线程进入无限循环状态，等待接收消息
}
```

其中做了2件事情：

1. 初始化主线程的Looper、主Handler
2. 调用ActivityThread.attach 方法，发出初始化Application的MSG

```java
//ActivityThread.java 
final ApplicationThread mAppThread = new ApplicationThread();
public void attach(boolean system){
    ...
 	//1.
    final IActivityManager mgr = ActivityManagerNative.getDefault();  
    try {
        //2.
        mgr.attachApplication(mAppThread);
    } catch (RemoteException ex) {
        throw ex.rethrowFromSystemServer();
    }
    ...
}
```

先看1.

```java
//ActivityManagerNative.java
static public IActivityManager getDefault() {
        return ActivityManager.getService();
 }
```

```java
//ActivityManager.java
public static IActivityManager getService() {
        return IActivityManagerSingleton.get();
    }

private static final Singleton<IActivityManager> IActivityManagerSingleton =
       new Singleton<IActivityManager>() {
           @Override
          protected IActivityManager create() {
              //IBinder实例是在这里获得的。
            	final IBinder b = ServiceManager.getService(Context.ACTIVITY_SERVICE);
             	final IActivityManager am = IActivityManager.Stub.asInterface(b);
                return am;
                }
            };
```

实际拿到的事一个单例的IActivityManager，IActivityManager实际是一个AIDL的interface，在ActivityThread跨进程调用初始化Application，具体的实现在ActivityManagerService中

下面进入ActivityManagerService中，

```java
//ActivityManagerService.java
public class ActivityManagerService extends IActivityManager.Stub
        implements Watchdog.Monitor, BatteryStatsImpl.BatteryCallback {
    
     public final void attachApplication(IApplicationThread thread, long startSeq) {
        synchronized (this) {
            ...
            attachApplicationLocked(thread, callingPid, callingUid, startSeq);
            ...
        }
    }
     private final boolean attachApplicationLocked(IApplicationThread thread, int pid) {
        ....
        //通过binder，跨进程调用ApplicationThread的bindApplication()方法, 下面代码逻辑重回ActivityThread.java
            thread.bindApplication(processName, ...);
        ....

    }
    
}
```

```java
//ActivityThread.java
private class ApplicationThread extends Binder implements IApplicationThread{

    ...
        public final void bindApplication(String processName,...) {
			...
        //发消息
            sendMessage(H.BIND_APPLICATION, data);
        }

    ...
        private void sendMessage(int what, Object obj, int arg1, int arg2, boolean async) {
        Message msg = Message.obtain();
        msg.what = what;
        msg.obj = obj;
        msg.arg1 = arg1;
        msg.arg2 = arg2;
        if (async) {
            msg.setAsynchronous(true);
        }
        //通过mH把BIND_APPLICATION消息发给H处理
        mH.sendMessage(msg);
        }
    ...

}
```

其中，H 是ActivityThread中的一个内部类，同时继承自Handler，使用H的目的是，把代码执行的逻辑从binder线程池里的线程切换到main线程里去执行。

```java
//ActivityThread.java
private class H extends Handler {
    ...
    public static final int BIND_APPLICATION        = 110;
    ...
    public void handleMessage(Message msg) {
        switch (msg.what) {
            ...
                case BIND_APPLICATION:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "bindApplication");
                    AppBindData data = (AppBindData)msg.obj;
                    handleBindApplication(data);//调用ActivityThread的handleBindApplication()方法处理
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
            ...
        }

    }
}
```

```java
//ActivityThread.java
public final class ActivityThread {
    ...
    private void handleBindApplication(AppBindData data) {
        ...
        //通过反射创建Instrumentation 对象
                java.lang.ClassLoader cl = instrContext.getClassLoader();
                mInstrumentation = (Instrumentation)
                    cl.loadClass(data.instrumentationName.getClassName()).newInstance();

        data.info = getPackageInfoNoCheck(data.appInfo, data.compatInfo);
        //创建app运行时的上下文对象，并对其进行初始化.
        final ContextImpl appContext = ContextImpl.createAppContext(this, data.info);
        appContext.init(data.info, null, this);
        //这里的data.info是LoadedApk类的对象
        //在这里创建了上层开发者的代码中所涉及的Applicaiton类的对象
        Application app = data.info.makeApplication(data.restrictedBackupMode, null);
        ...

        //如果有ContentProvider的话, 先加载ContentProvider，后调用Application的onCreate()方法
        List<ProviderInfo> providers = data.providers;
        if (providers != null) {
            installContentProviders(app, providers);
        }
        //调Application的生命周期函数 onCreate()
        mInstrumentation.callApplicationOnCreate(app);
        
    }
    ...

}
```

```java
// LoadedApk.java
public final class LoadedApk {
    ...
    public Application makeApplication(boolean forceDefaultAppClass,
            Instrumentation instrumentation) {
        Application app = null;
            app = mActivityThread.mInstrumentation.newApplication(
                    cl, appClass, appContext);
        return app;
    }
    ...

}
```

```java
// Instrumentation.java
public class Instrumentation {
    ...
    public void callApplicationOnCreate(Application app) {
        app.onCreate();
    }
    ...
}
```

调用步骤：

1. 入口是ActivityThread.main(), 创建UI线程的消息循环
2. binder -> ActivityManagerService#attachApplication
3. binder ->  ActivityThread#ApplicationThread#bindApplication 发送创建Application的Msg
4. Handler切换到main线程， 调用到 ActivityThread#handleBindApplication 中使用Instrumentation 创建Application 

## 总结：

1. 从ActivityThread.main()开始，创建UI线程的消息循环
2. 通过binder 经过 ActivityManagerService， 最终由ApplicationThread中使用Handler 发送创建Application的MSG
3. 由Instrumentation 创建Application 

## 细节补充

1. Application、Instrumentation 是反射创建
2. 使用Handler的作用是从binder线程池里的线程切换到main线程里去执行。

# Activity 启动

当Application初始化完成后，系统会更具Manifests中的配置的启动Activity发送一个Intent去启动相应的Activity。

其余情况，大多都是从Activity中的startActivityForResult 开始

## 1. 在APP进程发起startActivity

```java
//Activity.java  
public void startActivityForResult(@RequiresPermission Intent intent, int requestCode,
            @Nullable Bundle options) {
      		...
            Instrumentation.ActivityResult ar =
                mInstrumentation.execStartActivity(
                    this, mMainThread.getApplicationThread(), mToken, this,
                    intent, requestCode, options);
            ...
    }
```

```java
// Instrumentation.java
public ActivityResult execStartActivity(
            Context who, IBinder contextThread, IBinder token, Activity target,
            Intent intent, int requestCode, Bundle options) {
       		...
            int result = ActivityTaskManager.getService()
                .startActivity(whoThread, who.getBasePackageName(), intent,
                        intent.resolveTypeIfNeeded(who.getContentResolver()),
                        token, target != null ? target.mEmbeddedID : null,
                        requestCode, 0, null, options);
            checkStartActivityResult(result, intent);
            ...
        return null;
    }
```

```java
//ActivityTaskManager.java 
public static IActivityTaskManager getService() {
        return IActivityTaskManagerSingleton.get();
    }
   
private static final Singleton<IActivityTaskManager> IActivityTaskManagerSingleton =
       new Singleton<IActivityTaskManager>() {
                @Override
        protected IActivityTaskManager create() {
             final IBinder b = ServiceManager.getService(Context.ACTIVITY_TASK_SERVICE);
             return IActivityTaskManager.Stub.asInterface(b);
          }
      };
```

## 2. 在ATMS进程发起startActivity

```java
//com.android.server.wm.ActivityTaskManagerService
public class ActivityTaskManagerService extends IActivityTaskManager.Stub {
    // 方法重载，最终调了这个方法
   int startActivityAsUser(IApplicationThread caller, ...) {
		...
        return getActivityStartController().obtainStarter(intent, "startActivityAsUser")
                ...
            	// 这里，最后一步 调用的是ActivityStarter.execute()
                .execute();

    }
}
```

IActivityTaskManager.aidl就是Activity和ActivityTaskManagerService之间的桥梁

```java
//重点方法调用
//ActivityStarter#startActivityUnchecked()->
//RootActivityContainer#resumeFocusedStacksTopActivities()->
//ActivityStack#resumeTopActivityUncheckedLocked()
boolean resumeTopActivityUncheckedLocked(ActivityRecord prev, ActivityOptions options){
    ...
    boolean result = false;
    if (targetStack != null && (targetStack.isTopStackOnDisplay()
            || getTopDisplayFocusedStack() == targetStack)) {
        result = targetStack.resumeTopActivityUncheckedLocked(target, targetOptions);
    }
    ...
}


```

## 3. TopActivity-onPause()流程发起

```java
//ActivityStack#resumeTopActivityUncheckedLocked()-> 
//ActivityStack#resumeTopActivityUncheckedLocked-> 
//ClientLifecycleManager#scheduleTransaction()此处，操作对应生命周期的onPause流程，（生命周期指令从这里发出）
private boolean resumeTopActivityInnerLocked(ActivityRecord prev, ActivityOptions options) {
		...
            //栈顶的pause
        transaction.setLifecycleStateRequest(
              ResumeActivityItem.obtain(next.app.getReportedProcState(),
                getDisplay().mDisplayContent.isNextTransitionForward()));
                mService.getLifecycleManager().scheduleTransaction(transaction);
    	...
            //新建的create
            mStackSupervisor.startSpecificActivityLocked(next, true, false);
    	...
}
```

mService：ATMS 

mService.getLifecycleManager()：ClientLifecycleManager

mStackSupervisor：ActivityStackSupervisor

## 4. onCreate()流程发起

与onPause相同的方法中，栈顶Activity onPause流程后，紧接着就是新Activity的onCreate流程

```java
//ActivityStackSupervisor#realStartActivityLocked()
 boolean realStartActivityLocked(ActivityRecord r, WindowProcessController proc,
            boolean andResume, boolean checkConfig) throws RemoteException {
     ...
         // Schedule transaction.
	mService.getLifecycleManager().scheduleTransaction(clientTransaction);
     ...
 }

```

## 5. 安排事务到APP进程

```java
//ClientLifecycleManager#scheduleTransaction
void scheduleTransaction(ClientTransaction transaction) throws RemoteException {
    final IApplicationThread client = transaction.getClient();//ClientTransaction
    transaction.schedule();
}
```

## 6. 回到APP进程

```java
//ClientTransaction#schedule()->
//ActivityThread#ApplicationThread#scheduleTransaction()->
//ActivityThread#ClientTransactionHandler#scheduleTransaction()  通过H发送了EXECUTE_TRANSACTION消息
void scheduleTransaction(ClientTransaction transaction) {
    transaction.preExecute(this);
    sendMessage(ActivityThread.H.EXECUTE_TRANSACTION, transaction);
}

// ActivityThread#H handleMsg片段:
case EXECUTE_TRANSACTION:
    final ClientTransaction transaction = (ClientTransaction) msg.obj;
	// 此处判断执行什么流程
    mTransactionExecutor.execute(transaction);
    if (isSystem()) {
        // Client transactions inside system process are recycled on the client side
        // instead of ClientLifecycleManager to avoid being cleared before this
        // message is handled.
        transaction.recycle();
    }
    // TODO(lifecycler): Recycle locally scheduled transactions.
    break;
```

## 7. onPause()

如果对应的是onPause()流程,

```java
//TransactionExecutor#execute()->
//TransactionExecutor#executeLifecycleState()
private void executeLifecycleState(ClientTransaction transaction) {
    	//此处对象 即 PauseActivityItem，ActivityLifecycleItem为其父类
        final ActivityLifecycleItem lifecycleItem = transaction.getLifecycleStateRequest();
        ...
        lifecycleItem.execute(mTransactionHandler, token, mPendingActions);
        lifecycleItem.postExecute(mTransactionHandler, token, mPendingActions);
    }

//PauseActivityItem#execute
public void execute(ClientTransactionHandler client, IBinder token,
            PendingTransactionActions pendingActions) {
        Trace.traceBegin(TRACE_TAG_ACTIVITY_MANAGER, "activityPause");
    //此处的这个ActivityThread，其实就是ActivityThread
        client.handlePauseActivity(token, mFinished, mUserLeaving, mConfigChanges, pendingActions,
                "PAUSE_ACTIVITY_ITEM");
        Trace.traceEnd(TRACE_TAG_ACTIVITY_MANAGER);
    }

//ActivityThread#handlePauseActivity
public void handlePauseActivity(IBinder token, boolean finished, boolean userLeaving,
            int configChanges, PendingTransactionActions pendingActions, String reason) {
        	...
            performPauseActivity(r, finished, reason, pendingActions);
		   ...
    }
//ActivityThread#performPauseActivity->
//ActivityThread#performPauseActivityIfNeeded
  private void performPauseActivityIfNeeded(ActivityClientRecord r, String reason) {
   		...
        //Instrumentation中执行Activity的生命周期
        mInstrumentation.callActivityOnPause(r.activity);
        ...
}
//Instrumentation#callActivityOnPause
public void callActivityOnPause(Activity activity) {
    activity.performPause();
}
```

```java
//Activity#performPause
final void performPause() {
    dispatchActivityPrePaused();
    mDoReportFullyDrawn = false;
    mFragments.dispatchPause();
    mCalled = false;
    //Activity#onPause
    onPause();
    writeEventLog(LOG_AM_ON_PAUSE_CALLED, "performPause");
    mResumed = false;
    if (!mCalled && getApplicationInfo().targetSdkVersion
            >= android.os.Build.VERSION_CODES.GINGERBREAD) {
        throw new SuperNotCalledException(
                "Activity " + mComponent.toShortString() +
                " did not call through to super.onPause()");
    }
    dispatchActivityPostPaused();
}
```

## 8. onCreate()

如果是Create流程

```java
//TransactionExecutor#execute()->
//TransactionExecutor#executeCallbacks()->
public void executeCallbacks(ClientTransaction transaction) {
    	//此处对象 即 LaunchActivityItem，ClientTransaction为其父类
        item.execute(mTransactionHandler, token, mPendingActions);
    }

//LaunchActivityItem#execute()->
public void execute(ClientTransactionHandler client, IBinder token,
        PendingTransactionActions pendingActions) {
    Trace.traceBegin(TRACE_TAG_ACTIVITY_MANAGER, "activityStart");
    ActivityClientRecord r = new ActivityClientRecord(token, mIntent, mIdent, mInfo,
            mOverrideConfig, mCompatInfo, mReferrer, mVoiceInteractor, mState, mPersistentState,
            mPendingResults, mPendingNewIntents, mIsForward,
            mProfilerInfo, client, mAssistToken);
    client.handleLaunchActivity(r, pendingActions, null /* customIntent */);
    Trace.traceEnd(TRACE_TAG_ACTIVITY_MANAGER);
}

//ActivityThread#handleLaunchActivity()
public Activity handleLaunchActivity(ActivityClientRecord r,
            PendingTransactionActions pendingActions, Intent customIntent) {
    ...
     final Activity a = performLaunchActivity(r, customIntent);
    ...
}

//ActivityThread#performLaunchActivity
private Activity performLaunchActivity(ActivityClientRecord r, Intent customIntent) {
    	 ...
         java.lang.ClassLoader cl = appContext.getClassLoader();
   		 //还是通过Instrumentation 创建的Activity
         activity = mInstrumentation.newActivity(
                    cl, component.getClassName(), r.intent);
  		  ...
          //activity.attach
      	 activity.attach(appContext, this, getInstrumentation(), r.token,
                        r.ident, app, r.intent, r.activityInfo, title, r.parent,
                        r.embeddedID, r.lastNonConfigurationInstances, config,
                        r.referrer, r.voiceInteractor, window, r.configCallback,
                        r.assistToken);
    	  ...
              //activity.create
           if (r.isPersistable()) {
                    mInstrumentation.callActivityOnCreate(activity, r.state, r.persistentState);
                } else {
                    mInstrumentation.callActivityOnCreate(activity, r.state);
                }
    		....
}
```

调用Activity的performCreate()：

```java
//Activity#performCreate()->
//Activity#callActivityOnCreate
public void callActivityOnCreate(Activity activity, Bundle icicle,
        PersistableBundle persistentState) {
    prePerformCreate(activity);
    activity.performCreate(icicle, persistentState);
    postPerformCreate(activity);
}
```

```java
//Activity#performCreate
final void performCreate(Bundle icicle, PersistableBundle persistentState) {
    dispatchActivityPreCreated(icicle);
    mCanEnterPictureInPicture = true;
    restoreHasCurrentPermissionRequest(icicle);
    //Activity#onCreate
    if (persistentState != null) {
        onCreate(icicle, persistentState);
    } else {
        onCreate(icicle);
    }
    writeEventLog(LOG_AM_ON_CREATE_CALLED, "performCreate");
    mActivityTransitionState.readState(icicle);

    mVisibleFromClient = !mWindow.getWindowStyle().getBoolean(
            com.android.internal.R.styleable.Window_windowNoDisplay, false);
    mFragments.dispatchActivityCreated();
    mActivityTransitionState.setEnterActivityOptions(this, getActivityOptions());
    dispatchActivityPostCreated(icicle);
}
```

## 总结：

1. Activity 发起请求，Binder 到ATMS 处理请求，接着就会进行一些校验和判断权限，包括进程检查，intent检查，权限检查等，根据启动模式计算 flag ，设置启动 Activity 的 Task 栈。
2. 检查要启动的 Activity 进程是否存在，存在则向客户端进程 ApplicationThread 回调启动 Activity，否则就创建进程。
   1. 进程存在
      - 通过ActivityStack、ClientLifecycleManager 发出生命周期的指令，先发起TopActivity的Pause流程
      - 经由ActivityThread 中的Handler，切换到主线程中，在TransactionExecutor#execute() 中执行Activity的onPause
   2. 进程不存在，通过Zygote fork出应用进程
3. 通过TransactionExecutor#execute() 中执行 由Instrumentation反射创建的新Activity 对象的onCreate



# APK打包

APK包含文件：

- AndroidManifest.xml 程序全局配置文件
- classes.dex Dalvik字节码
- resources.arsc 资源索引表, 解压缩resources.ap_就能看到
- res\ 该目录存放资源文件(图片，文本，xml布局)
- assets\ 该目录可以存放一些配置文件
- src\ java源码文件
- libs\ 存放应用程序所依赖的库
- gen\ 编译器根据资源文件生成的java文件
- bin\ 由编译器生成的apk文件和各种依赖的资源
- META-INF\ 该目录下存放的是签名信息

![](https://pic3.zhimg.com/80/v2-2227b8b9b548a55c65cad493319a2d06_1440w.webp)

### **1. 打包资源文件，生成R.java文件**

打包资源的工具是aapt（The Android Asset Packaing Tool）

在这个过程中，项目中的AndroidManifest.xml文件和布局文件XML都会编译，然后生成相应的R.java，另外AndroidManifest.xml会被aapt编译成二进制。

存放在APP的res目录下的资源，该类资源在APP打包前大多会被编译，变成二进制文件，并会为每个该类文件赋予一个resource id。**对于该类资源的访问，应用层代码则是通过resource id进行访问的**。Android应用在编译过程中aapt工具会对资源文件进行编译，并生成一个resource.arsc文件，resource.arsc文件相当于一个文件索引表，记录了很多跟资源相关的信息。

### **2. 处理aidl文件，生成相应的.Java文件**

这一过程中使用到的工具是aidl（Android Interface Definition Language），即Android接口描述语言。

aidl工具解析接口定义文件然后生成相应的.Java代码接口供程序调用。

如果在项目没有使用到aidl文件，则可以跳过这一步。

### **3. 编译项目源代码，生成class文件**

项目中所有的.Java代码，包括*R.java*和*.aidl*文件，都会变Java编译器（javac）编译成*.class*文件，生成的class文件位于工程中的bin/classes目录下。

### **4. 转换所有的class文件，生成classes.dex文件**

dx工具生成可供Android系统Dalvik虚拟机执行的classes.dex文件。

任何第三方的*libraries*和*.class*文件都会被转换成*.dex*文件。

dx工具的主要工作是将Java字节码转成成Dalvik字节码、压缩常量池、消除冗余信息等。

### **5. 打包生成APK文件**

所有没有编译的资源，如images、assets目录下资源（**该类文件是一些原始文件，APP打包时并不会对其进行编译，而是直接打包到APP中，对于这一类资源文件的访问，应用层代码需要通过文件名对其进行访问**）；

编译过的资源和*.dex*文件都会被apkbuilder工具打包到最终的*.apk*文件中。

打包的工具apkbuilder位于 android-sdk/tools目录下。

### **6. 对APK文件进行签名**

一旦APK文件生成，它必须被签名才能被安装在设备上。

在开发过程中，主要用到的就是两种签名的keystore。一种是用于调试的debug.keystore，它主要用于调试，在Eclipse或者Android Studio中直接run以后跑在手机上的就是使用的debug.keystore。

另一种就是用于发布正式版本的keystore。

### **7. 对签名后的APK文件进行对齐处理**

如果你发布的apk是正式版的话，就必须对APK进行对齐处理，用到的工具是zipalign。

对齐的主要过程是将APK包中所有的资源文件距离文件起始偏移为4字节整数倍，这样通过内存映射访问apk文件时的速度会更快。对齐的作用就是减少运行时内存的使用。



# Android ClassLoader

Android中的ClassLoader是一个抽象类，使用的是：

1. DexClassLoader：可以加载jar/apk/dex，可以从SD卡中加载未安装的apk

2. PathClassLoader：要传入系统中apk的存放Path，所以只能加载已安装的apk文件

**加载过程**：

当前ClassLoader实例是否加载过此类

- 有： 直接返回

- 没有：查询Parent是否已经加载过此类（递归）
  - 有：返回
  - 没有：Child执行类的加载

## 细节

1. **一个运行的Android应用至少有2个ClassLoader**：
   - BootClassLoader（系统启动的时候创建的）
   - PathClassLoader（应用启动时创建的，用于加载“/data/app/me.kaede.anroidclassloadersample-1/base.apk”里面的类）

# Android虚拟机

| 虚拟机                 | 特点                                                         | 版本            |
| ---------------------- | ------------------------------------------------------------ | --------------- |
| Dalvik                 | 即时编译器（JIT，just in time）：执行的时候编译+运行，安装比较快，开启应用比较慢，应用占用空间小 | android 5.0之前 |
| ART（Android Runtime） | 预编译（AOT，Ahead-Of-Time）：应用在第一次安装的时候，字节码就会预先编译成机器码，占用空间较大（10%-20%） | android 5.0之后 |

Android 7.0，JIT 编译器回归，形成 AOT/JIT 混合编译模式，这种混合编译模式的特点是：

- 应用在安装的时候 dex 不会被编译
- 应用在运行时 dex 文件先通过解析器（Interpreter）后会被直接执行（这一步骤跟 Android 2.2 - Android 4.4之前的行为一致），与此同时，热点函数（Hot Code）会被识别并被 JIT 编译后存储在 jit code cache 中并生成 profile 文件以记录热点函数的信息。
- 手机进入 IDLE（空闲） 或者 Charging（充电） 状态的时候，系统会扫描 App 目录下的 profile 文件并执行 AOT 过程进行编译。



# [AOP思想](https://juejin.im/post/6844904017760370695)

从织入的时机的角度看，可以分为源码阶段、class阶段、dex阶段、运行时织入。

**源码阶段、class阶段、dex织入**，由于他们都发生在class加载到虚拟机前，我们统称为静态织入， 而在运行阶段发生的改动，我们统称为动态织入。

常见的技术框架如下表：

| 织入时机 | 技术框架                      |
| :------: | ----------------------------- |
| 静态织入 | APT，AspectJ、ASM、Javassit   |
| 动态织入 | java动态代理，cglib、Javassit |

静态织入发生在编译器，因此几乎不会对运行时的效率产生影响；动态织入发生在运行期，可直接将字节码写入内存，并通过反射完成类的加载，所以效率相对较低，但更灵活。

动态织入的前提是类还未被加载，你不能将一个已经加载的类经过修改再次加载，这是ClassLoader的限制。但是可以通过另一个ClassLoader进行加载，虚拟机允许两个相同类名的class被不同的ClassLoader加载，在运行时也会被认为是两个不同的类，因此需要注意不能相互赋值， 不然会抛出ClassCastException。



## APT

**APT**（Annotation Processing Tool）即注解处理器，在Gradle 版本>=2.2后被annotationProcessor取代。

它用来在编译时扫描和处理注解，扫描过程可使用[ auto-service ](https://github.com/google/auto/tree/master/service)来简化寻找注解的配置，在处理过程中可生成java文件（创建java文件通常依赖[ javapoet ](https://github.com/square/javapoet)这个库）。常用于生成一些模板代码或运行时依赖的类文件，比如常见的ButterKnife、Dagger、ARouter，它的优点是简单方便。

APT技术的不足，通常只是用来创建新的类，而不能对原有类进行改动，在不能改动的情况下，只能通过反射实现动态化。

## AspectJ

实现代码织入有两种方式，一是自行编写.ajc文件，二是使用AspectJ提供的@Aspect、@Pointcut等注解，二者最终都是通过ajc编译器完成代码的织入。

**使用：**

- @Aspect 用它声明一个类，表示一个需要执行的切面。
- @Pointcut 声明一个切点。
- @Before/@After/@Around/...（统称为Advice类型） 声明在切点前、后、中执行切面代码。

**存在的问题：**

1. 重复织入、不织入

   假如想对Activity生命周期织入埋点统计，我们可能写出这样的切点代码。

   ```java
   @Pointcut("execution(* android.app.Activity+.on*(..))")
   public void callMethod() {}
   ```

   由于Activity.class不参与打包（android.jar位于android设备内），参与打包是那些支持库比如support-v7中的AppCompatActivity，还有项目里定义的Activity，这就导致：

   1. 如果业务Activity中如果没有复写生命周期方法将不会织入。
   2. 如果的Activity继承树上如果都复写了生命周期方法，那么继承树上的所有Activity都会织入统计代码，这会导致重复统计。

   解决办法是项目内定义一个基类Activity（比如BaseActivity），然后复写所有生命周期方法，然后将切点代码精确到这个BaseActivity。

   ```java
   @Pointcut("execution(* com.xxx.BaseActivity.on*(..))")
   public void callMethod() {}
   ```

   但如果真这样做的话，你肯定会反问还需要AspectJ做什么...

2. 出问题难排查

   这是AOP技术的实现方式决定的，修改字节码过程，对上层应用无感知，容易将问题隐藏，排查难度大。因此如果项目中使用了AOP技术应当完善文档，并知会协同开发人员。

3. 编译时间变长

   Transform过程，会遍历所有class文件，查找符合需求的切入点，然后插入字节码。如果项目较大且织入代码较多，会增加十几秒左右的编译时间。

   如前文提到的，有两种办法解决这个问题：

   1. 使用exclude过滤掉不需要执行织入的包名。
   2. 如果织入代码在debug环境不需要织入，比如埋点，则使用enabled false 关闭AspectJ功能。

4. 兼容性

   如果使用的三方库也使用了AspectJ，可能导致未知的风险。

   比如sample项目中同时使用Hugo，会导致工程中的class不会被打入APK中，运行时会出现ClassNotFoundException。这可能是Hugo项目编写的Plugin插件与Hujiang的AspectJX插件有冲突导致的。

[Android 函数耗时统计工具之Hugo](https://juejin.im/post/6844903945186476040)

## ASM

ASM是一个字节码操作框架，可用来动态生成字节码或者对现有的类进行增强。ASM可以直接生成二进制的class字节码，也可以在class被加载进虚拟机前动态改变其行为，比如方法执行前后插入代码，添加成员变量，修改父类，添加接口等等。

> Matrix是微信开源的一个APM框架，其中TraceCanary子模块用于监测帧率低、卡顿、ANR等场景，具备函数耗时统计的功能。
>
> 为了实现函数的耗时统计，通常的做法都是在函数执行开始和结束为止进行插桩，最后以两个插桩点的时间差为函数的执行时间。

**ASM优点：**

- 由于直接操作的是字节码，因此相比其他框架效率更高。
- 从ASM5开始已经支持Java8的部分语法，比如lamabda表达式。
- 因为ASM偏向底层，很多其他的上层框架也以ASM作为其底层操作字节码的技术栈，比如Groovy、cglib。

**ASM不足：**

1. 过滤类和方法需要硬编码，且不够灵活，需要对插件进行二次封装，而在AspectJ中已经封装好了切面表达式。
2. 很难实现在方法调用前后织入新的代码，而在AspectJ中一个call关键字就解决了。
3. 如果需要在指定的类，指定的方法中织入代码，需要编写相应的过滤条件，这也是相比于AspectJ而言不太方便的地方，AspectJ可通过声明切面注解完成精准的织入。



## javassit

javassit是一个开源的字节码创建、编辑类库，现属于Jboss web容器的一个子模块，特点是简单、快速，与AspectJ一样，使用它不需要了解字节码和虚拟机指令，这里是[官方文档](https://www.javassist.org/tutorial/tutorial.html)。

javassit核心的类库包含ClassPool，CtClass ，CtMethod和CtField。

- ClassPool：一个基于HashMap实现的CtClass对象容器。
- CtClass：表示一个类，可从ClassPool中通过完整类名获取。
- CtMethods：表示类中的方法。
- CtFields ：表示类中的字段。

## 总结

| 技术框架     | 特点                                                         | 开发难度 | 优势                                                     | 不足                                       |
| ------------ | ------------------------------------------------------------ | -------- | -------------------------------------------------------- | ------------------------------------------ |
| APT          | 常用于通过注解减少模板代码，对类的创建于增强需要依赖其他框架。 | ★★       | 开发注解简化上层编码。                                   | 使用注解对原工程具有侵入性。               |
| AspectJ      | 提供完整的面向切面编程的注解。                               | ★★       | 真正意义的AOP，支持通配、继承结构的AOP，无需硬编码切面。 | 重复织入、不织入问题                       |
| ASM          | 面向字节码指令编程，功能强大。                               | ★★★      | 高效，ASM5开始支持java8。                                | 切面能力不足，部分场景需硬编码。           |
| Javassit     | API简洁易懂，快速开发。                                      | ★        | 上手快，新人友好，具备运行时加载class能力。              | 切点代码编写需注意class path加载问题。     |
| java动态代理 | 运行时扩展代理接口功能。                                     | ★        | 运行时动态增强。                                         | 仅支持代理接口，扩展性差，使用反射性能差。 |



# 热修复

## 方案对比

![](https://img1.imgtp.com/2023/09/14/x6GFj3tp.jpg)

## 微博使用-Robust

[参考链接1](https://zhuanlan.zhihu.com/p/630582550)

[参考链接2](https://tech.meituan.com/2016/09/14/android-robust.html)

大致流程图：

![](https://awps-assets.meituan.net/mit-x/blog-images-bundle-2016/286c9718.png)



### 打包阶段

自定义Transform，使用字节码操作工具 （robust使用了 javaassit 和 ASM 框架），在每个方法前插入一段类型为ChangeQuickRedirect的静态变量。

![](https://img1.imgtp.com/2023/09/14/flo9GRtT.png)





### 生成补丁

补丁包主要类文件：

- PatchesInfoImpl


- xxxPatchControl


- xxxPatch

![](https://img1.imgtp.com/2023/09/14/oHsUBm7j.png)

### 加载补丁

1. 开启一个子线程到指定路径去读 patch 包。
2. 读取补丁包后，新建 DexClassLoader 去加载补丁 dex 文件。
3. 反射得到 PatchesInfoImpl.class，并创建其对象。
4. 调用 getPatchedClassesInfo( ) 方法得到需要补丁的类信息。
5. 根据信息反射拿到每个修改类在当前环境中的的 class。
6. 反射修改 class 里类型为 ChangeQuickRedirect 的静态变量，将其修改为 xxxPatchControl 这个类new出来的对象。

### 特殊处理

#### 混淆

先针对混淆前的代码生成patch.class，然后利用生成release包时对应的mapping文件中的class的映射关系，对patch.class做字符串上的处理，让它使用线上运行环境中混淆的class。

#### this

原因：在补丁的xxPatch类中，this指代的是xxPatch类对象本身，而过程中实际想要的对象是被补丁类的对象

处理：补丁的xxPatch类构造方法中，传递xx类对象。出现this时，使用xx类对象调用。

#### super

原因： 在Java中super是个关键字，也无法通过别的对象来访问到。

通过对class文件做分析，发现普通的函数调用是使用JVM指令集的invokevirtual指令，而super.onCreate的调用使用的是invokesuper指令，

处理：

1. 使用Smali（Android逆向工具，dex文件反编译之后就是Smali代码，Smali语言是Android虚拟机的反汇编语言）修改指令为invokesuper指令。

2. 补丁类也要继承自xx类的父类，否则会报错找不到该方法。

#### 内联

如果线上包中不存在（上次构建过程中被移除或者被内联），补丁生成阶段需要当做新增类/方法加进来

#### 可见性

如果线上包中不可以被外部访问（上次构建过程中 public 被改为 private），补丁生成阶段需要将直接调用改成反射调用

### 优化

**自动补丁 - generatPatch流程**：

1. 识别被优化过的代码

   调用InlineClassFactory.dealInLineClass(...)方法识别被优化过的方法并记录缓存；

   这里的优化是泛指，包括被优化、内联、新增过的类和方法。具体的逻辑为扫描修改后的类和方法，如果这些类和方法不在 mapping 文件中存在，那么可以定义为被优化过；

2. 识别分析是否包含super方法
   initSuperMethodInClass() 负责，分析所有需要修改的类及方法是否含有super信息，并缓存记录；

3. 生成 xxxPatch 类
   调用 PatchesFactory.createPatch 反射翻译修改后的代码，生成xxxPatch类
   是整个自动化流程最核心的地方 （处理 this, super, 内联等具体逻辑）

4. 生成 xxxPatchControl 类

5. 生成 PatchInfoImpl 类

**增加包大小及潜在的66536问题**

1. robust.xml中的一些开关选项 （可指定需要/不需要插桩的类名）
2. 规避接口和无方法类
3. 规避抽象方法
4. 规避native方法
5. 规避接口及简单方法（set / get / 1行代码）

### 微博的针对的修改

1. 微博修复so 
   hook DexPatchList 的 nativeLibraryDirectories对象。

2. 微博修复资源文件
   替换ResourcesManager中所有Resources对象中AssetManager

