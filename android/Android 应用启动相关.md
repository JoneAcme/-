**Android 应用启动相关**



[toc]





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

2. **Zygote跨进程通信之所以用socket而不是binder，是因为binder通信是多线程的，而Zygote需要在单线程状态下fork子进程来避免死锁问题。**

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

1. AMS向Zygote发起启动应用的socket请求，Zygote收到请求fork出进程，返回进程的pid给AMS；
2. 应用进程启动好后，执行入口main函数，通过attachApplication方法告诉AMS已经启动，同时传入应用进程的binder句柄IApplicationThread。

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

## 3. onPause()流程发起

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

1. Activity 发起请求，Binder 到ATMS 处理请求，
2. 通过ActivityStack、ClientLifecycleManager 发出生命周期的指令，先发起TopActivity的Pause流程
3.  经由ActivityThread 中的Handler，切换到主线程中，在TransactionExecutor#execute() 中执行Activity的onPause
4.  同样通过TransactionExecutor#execute() 中执行 由Instrumentation反射创建的新Activity 对象的onCreate

