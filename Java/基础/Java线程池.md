# Java线程池



# Java线程池

[TOC]

## 1. 简介



![img](https:////upload-images.jianshu.io/upload_images/944365-ab8efb01ded53bbb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000)

示意图

------

## 2. 工作原理

### 2.1 核心参数

- 线程池中有6个核心参数，具体如下

![](https:////upload-images.jianshu.io/upload_images/944365-0a2b6ccbc2bd20f3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/970)

- **上述6个参数的配置 决定了 线程池的功能**，具体设置时机 = 创建 线程池类对象时 传入

> 1. `ThreadPoolExecutor`类 = 线程池的真正实现类
> 2. 开发者可根据不同需求 配置核心参数，从而实现自定义线程池

```
// 创建线程池对象如下
// 通过 构造方法 配置核心参数
   Executor executor = new ThreadPoolExecutor( 
                                              CORE_POOL_SIZE,
                                              MAXIMUM_POOL_SIZE,
                                              KEEP_ALIVE,
                                              TimeUnit.SECONDS, 
                                              sPoolWorkQueue,
                                              sThreadFactory 
                                               );

// 构造函数源码分析
    public ThreadPoolExecutor (int corePoolSize,
                               int maximumPoolSize,
                               long keepAliveTime,
                               TimeUnit unit,
                               BlockingQueue<Runnable workQueue>,
                               ThreadFactory threadFactory )
```

注：`Java`里已内置4种常用的线程池（即 已经配置好核心参数），下面会详细说明

### 2.2 内部原理逻辑

当线程池运行时，遵循以下工作逻辑



![img](https:////upload-images.jianshu.io/upload_images/944365-90cfd4951a587ebd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000)

示意图

------

## 3. 使用流程

线程池的使用流程如下

```
// 1. 创建线程池
   // 创建时，通过配置线程池的参数，从而实现自己所需的线程池
   Executor threadPool = new ThreadPoolExecutor(
                                              CORE_POOL_SIZE,
                                              MAXIMUM_POOL_SIZE,
                                              KEEP_ALIVE,
                                              TimeUnit.SECONDS,
                                              sPoolWorkQueue,
                                              sThreadFactory
                                              );
    // 注：在Java中，已内置4种常见线程池，下面会详细说明

// 2. 向线程池提交任务：execute（）
    // 说明：传入 Runnable对象
       threadPool.execute(new Runnable() {
            @Override
            public void run() {
                ... // 线程执行任务
            }
        });

// 3. 关闭线程池shutdown() 
  threadPool.shutdown();
  
  // 关闭线程的原理
  // a. 遍历线程池中的所有工作线程
  // b. 逐个调用线程的interrupt（）中断线程（注：无法响应中断的任务可能永远无法终止）

  // 也可调用shutdownNow（）关闭线程：threadPool.shutdownNow（）
  // 二者区别：
  // shutdown：设置 线程池的状态 为 SHUTDOWN，然后中断所有没有正在执行任务的线程
  // shutdownNow：设置 线程池的状态 为 STOP，然后尝试停止所有的正在执行或暂停任务的线程，并返回等待执行任务的列表
  // 使用建议：一般调用shutdown（）关闭线程池；若任务不一定要执行完，则调用shutdownNow（）
```

------

## 4. 常见的4类功能线程池

根据参数的不同配置，`Java`中最常见的线程池有4类：

- 定长线程池（`FixedThreadPool`）
- 定时线程池（`ScheduledThreadPool`）
- 可缓存线程池（`CachedThreadPool`）
- 单线程化线程池（`SingleThreadExecutor`）

> 即 对于上述4类线程池，`Java`已根据 应用场景 配置好核心参数

### 4.1 定长线程池（FixedThreadPool）

- 特点：只有核心线程 & 不会被回收、线程数量固定、任务队列无大小限制（超出的线程任务会在队列中等待）
- 应用场景：控制线程最大并发数
- 具体使用：通过 *Executors.newFixedThreadPool()*创建
- 示例：

```
// 1. 创建定长线程池对象 & 设置线程池线程数量固定为3
ExecutorService fixedThreadPool = Executors.newFixedThreadPool(3);

// 2. 创建好Runnable类线程对象 & 需执行的任务
Runnable task =new Runnable(){
  public void run(){
    System.out.println("执行任务啦");
     }
    };
        
// 3. 向线程池提交任务：execute（）
fixedThreadPool.execute(task);
        
// 4. 关闭线程池
fixedThreadPool.shutdown();
```

### 4.2 定时线程池（ScheduledThreadPool ）

- 特点：核心线程数量固定、非核心线程数量无限制（闲置时马上回收）
- 应用场景：执行定时 / 周期性 任务
- 使用：通过*Executors.newScheduledThreadPool()*创建
- 示例：

```
// 1. 创建 定时线程池对象 & 设置线程池线程数量固定为5
ScheduledExecutorService scheduledThreadPool = Executors.newScheduledThreadPool(5);

// 2. 创建好Runnable类线程对象 & 需执行的任务
Runnable task =new Runnable(){
       public void run(){
              System.out.println("执行任务啦");
          }
    };
// 3. 向线程池提交任务：schedule（）
scheduledThreadPool.schedule(task, 1, TimeUnit.SECONDS); // 延迟1s后执行任务
scheduledThreadPool.scheduleAtFixedRate(task,10,1000,TimeUnit.MILLISECONDS);// 延迟10ms后、每隔1000ms执行任务

// 4. 关闭线程池
scheduledThreadPool.shutdown();
```

### 4.3 可缓存线程池（CachedThreadPool）

- 特点：只有非核心线程、线程数量不固定（可无限大）、灵活回收空闲线程（具备超时机制，全部回收时几乎不占系统资源）、新建线程（无线程可用时）

> 任何线程任务到来都会立刻执行，不需要等待

- 应用场景：执行大量、耗时少的线程任务
- 使用：通过*Executors.newCachedThreadPool()*创建
- 示例：

```
// 1. 创建可缓存线程池对象
ExecutorService cachedThreadPool = Executors.newCachedThreadPool();

// 2. 创建好Runnable类线程对象 & 需执行的任务
Runnable task =new Runnable(){
  public void run(){
        System.out.println("执行任务啦");
            }
    };

// 3. 向线程池提交任务：execute（）
cachedThreadPool.execute(task);

// 4. 关闭线程池
cachedThreadPool.shutdown();

//当执行第二个任务时第一个任务已经完成
//那么会复用执行第一个任务的线程，而不用每次新建线程。
```

### 4.4 单线程化线程池（SingleThreadExecutor）

- 特点：只有一个核心线程（保证所有任务按照指定顺序在一个线程中执行，不需要处理线程同步的问题）
- 应用场景：不适合并发但可能引起IO阻塞性及影响UI线程响应的操作，如数据库操作，文件操作等
- 使用：通过*Executors.newSingleThreadExecutor()*创建
- 示例：

```
// 1. 创建单线程化线程池
ExecutorService singleThreadExecutor = Executors.newSingleThreadExecutor();

// 2. 创建好Runnable类线程对象 & 需执行的任务
Runnable task =new Runnable(){
  public void run(){
        System.out.println("执行任务啦");
            }
    };

// 3. 向线程池提交任务：execute（）
singleThreadExecutor.execute(task);

// 4. 关闭线程池
singleThreadExecutor.shutdown();
```

## 5 常见线程池 总结 & 对比



![img](https:////upload-images.jianshu.io/upload_images/944365-5d6a2497f809d62a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000)