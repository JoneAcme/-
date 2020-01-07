# JVM内存结构

# JVM：图文解析 Java内存结构



![img](https:////upload-images.jianshu.io/upload_images/944365-9c6e57ba45638191.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/772)

示意图

------

# 1. 内存模型 & 分区

- `Java`虚拟机在运行`Java`程序时，会管理着一块内存区域：运行时数据区
- 在运行时数据区里，会根据用途进行划分：
  1. `Java`虚拟机栈（栈区）
  2. 本地方法栈
  3. `Java`堆（堆区）
  4. 方法区
  5. 程序计数器



![img](https:////upload-images.jianshu.io/upload_images/944365-f370b46f0db07bee.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/790)



------

# 2. Java堆





![img](https:////upload-images.jianshu.io/upload_images/944365-621f236986b60a5a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/790)



- 简介



![img](https:////upload-images.jianshu.io/upload_images/944365-85f387b2b0ad0459.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000)

------

# 3. Java虚拟机栈





![img](https:////upload-images.jianshu.io/upload_images/944365-bfd6e6dd30d972f0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/790)

示意图

- 简介



![img](https:////upload-images.jianshu.io/upload_images/944365-9d4e3597129e40da.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000)



------

# 4. 本地方法栈



![img](https:////upload-images.jianshu.io/upload_images/944365-89fd4674668113b9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/790)

示意图

- 简介
  十分类似`Java`虚拟机栈，与Java虚拟机区别在于：服务对象，即
  Java虚拟机栈为执行 `Java`方法服务；本地方法栈为执行 `Native`方法服务

------

# 5. 方法区



![img](https:////upload-images.jianshu.io/upload_images/944365-065a5e781bcbe6be.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/790)

示意图

- 简介



![img](https:////upload-images.jianshu.io/upload_images/944365-e9e0581c451d306d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000)

示意图

- 注
  其内部包含一个运行时常量池，具体介绍如下：



![img](https:////upload-images.jianshu.io/upload_images/944365-e5ce7c7ffcfbb7df.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000)

示意图

------

# 6. 程序计数器

![img](https:////upload-images.jianshu.io/upload_images/944365-3f00e8ef07c0fcce.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/790)

示意图

- 简介



![img](https:////upload-images.jianshu.io/upload_images/944365-defa95a0c9ab5075.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000)

示意图

------

# 7. 额外知识：直接内存

- 定义：`NIO`类（`JDK`1.4引入）中基于通道和缓冲区的`I/O`方式 通过使用`Native`函数库 直接分配 的堆外内存
- 特点：不受堆大小限制

> 不属于虚拟机运行时数据区的一部分 & 不在堆中分配

- 应用场景：适用于频繁调用的场景

> 通过一个 存储在`Java`堆中的`DirectByteBuffer`对象 作为这块内存的引用 进行操作，从而避免在 `Java`堆和 `Native`堆之间来回复制数据，提高使用性能

- 抛出的异常：`OutOfMemoryError`，即与其他内存区域的总和 大于 物理内存限制

------

# 8. 总结



![img](https:////upload-images.jianshu.io/upload_images/944365-1c66953200e8253d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000)



链接：https://www.jianshu.com/p/6e2bc593f31c