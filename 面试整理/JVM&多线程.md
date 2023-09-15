**面试题知识点--java相关**



[TOC]



 

## JVM相关

### JVM内存模型

#### 程序计数器(线程私有)

当前线程所执行的字节码行号指示器。

 

#### java虚拟机栈(线程私有)

描述的是Java方法执行的内存模型：每个方法在执行的同时都会创建一个栈帧，用于存储局部变量表，操作数栈、动态链接、方法出口等信息。每个方法从调用直至执行完成的过程，就对应一个栈帧再虚拟机栈中入栈到出栈的过程。

Java虚拟机规范中，对这个区域规定了两种异常的状况：

1. 如果线程请求的栈深度大于虚拟机允许的深度，将抛出StackOverflowError

2. 如果虚拟机栈可以动态扩展，如果扩展时无法申请到足够的内存，就会抛出OutOfMemoryError异常

 

#### 本地方法栈(线程私有)

与虚拟机栈发挥的作用是非常相似的，它们之间的区别不过是：

虚拟机栈为虚拟机执行Java方法（字节码）服务。

本地方法栈则为虚拟机使用到的Native方法服务。

 

#### java堆

Java堆是被所有线程共享的一块内存区域，在虚拟机创建时创建。此内存区域唯一的目的就是存放对象实例。所有的对象实例以及数组都要在堆上分配。

 

#### 方法区

所有线程共享的内存区域，用于存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码数据等。

相对而言，垃圾收集行为在这个区域是较少出现的，但并非数据进入了方法区就如永久代的名字一样，永久存在了，这个区域的内存回收目标主要是针对常量池的回收和对类型的卸载。

当方法去无法满足内存分配需求时，会抛出OutOfMemoryError异常

 

#### 运行时常量池

是方法区的一部分，出了保存Class文件中描述的符号引用外，还会把翻译出来的直接引用也存储在运行常量池中。

运行时常量池相对于Class文件常量池的另外一个重要特征是具备动态性，运行期间也可能将新的常量放入池中。

 

#### 直接内存(虚拟机外内存)

并不是虚拟机运行时的一部分，也不是Java虚拟机规范中定义的内存区域。

使用：1.4中加入的NIO（New Input/Output）中，引入了一种基于通道（Channel）与缓冲区（Buffer）的I/O 方式，它可以使用Native函数库直接分配堆外内存然后通过存储在Java堆中的DirectByteBuffer对象作为这块内存的引用进行操作，避免在Java堆和Native堆中来回复制数据，能显著提高性能。

### 垃圾回收

#### 引用计数法

对象中添加一个引用计数器，每当有一个地方引用它时，计数器+1；当引用失效时，计数器-1。任何时刻计数器为0的对象就是不可能再被使用的。

缺点：相互引用着对方，导致引用计数都不为0，无法通知GC收集器回收它们。

#### 可达性分析法

通过一系列成为GC Roots的对象作为起始点，从这些节点开始向下搜索，搜索走过的路径成为引用链，当一个对象到GC Roots没有任何引用链相连接时，则证明此对象是不可用的，被判定为可回收对象。

可作为GC Roots的对象包括：

1.     虚拟机栈（栈帧中的本地变量表）中引用的对象

2.     方法区中静态属性引用的对象

3.     方法区中常量引用的对象

4.     本地方法栈中JNI（Native方法）引用的对象

#### Java中的四种引用

强引用 > 软引用 > 弱引用

| 引用类型                   | 说明                                                         |
| -------------------------- | ------------------------------------------------------------ |
| StrongReferenc（强引用）   | 普遍存在的，直接New出来的，只要强引用还在，垃圾回收器永远不会回收掉被引用的对象 |
| WeakReference （弱引用）   | 不管内存是否足够，只要发现，就会被回收。                     |
| SoftReference（软引用）    | 内存发生溢出之前，才会回收。                                 |
| PhantomReference（虚引用） | 一个对象是否有虚引用的存在，完全不会对其生存时间构成影响，也无法通过虚引用来取得一个对象的实例。为一个对象设置虚引用的唯一目的就是能在这个对象被收集器回收时，收到一个系统通知。 |

#### 回收方法区

可以不要求虚拟机在方法区实现垃圾收集，毕竟方法区进行垃圾收集的性价比一般较低。

​       永久代的垃圾收集主要回收两部分内容：废弃常量和无用的类。

​       废弃常量：是否有引用。

​       无用的类（同时满足以下三个条件）：

1.     该类所有的实例都已经被回收，Java堆中不存在该类的任何实例

2.     加载该类的ClassLoader已经被回收

3.     该类对应的java.lang.Class对象没有再任何地方被引用，无法在任何地方通过反射访问该类的方法

注：这里说的仅仅是可以，而并不是和对象一样，不使用了就必然会被回收

 

#### 标记-清除

步骤：

1.     首先标记处所有需要回收的对象

2.     在标记完成后统一回收所有被标记的对象

不足：

1．   效率问题。标记和清除两个过程的效率都不高

2．   空间问题。标记清除后悔产生大量不连续的内存碎片，导致以后在程序运行过程中需要分配较大对象时，无法找到足够连续的内存，而不得不提前出发另一次垃圾收集动作

#### 复制算法

将内存区域划分为两部分，每次只使用其中的一块，当这一块快用完了，就将还存活的对象复制到另一块内存上，然后把当前使用过的这块一次性清理掉。

优点：

不用考虑内存碎片等复杂情况，只要移动堆顶指针，按顺序分配内存即可

​       缺点：

​              内存使用率缩减

 

**复制算法优化**：

将内存分为一块较大的Eden空间和两块较小的Survivor空间，每次使用Eden和其中一块Survivor，当回收时，将Eden和Survivor中还存活着的对象一次性地复制到另一块Survivor空间上，最后清理掉Eden和用过的Survivor空间。如今默认的Eden和Survivor对象大小比例是8:1。

不能保证每次回收都只有不多于10%的对象存货，当Survivor空间不够用时，需要依赖其他内存（这里指老年代）进行分配担保。

当另一块Survivor空间没有足够的空间存放上一次新生代收集下来存货对象时，这些对象将直接通过分配担保进入老年代。

#### 标记-整理

​       标记整理算法过程仍与标记清除算法一样，但后续步骤不是直接对可回收的对象进行清理，而是让所有存活的对象都向一端移动，然后清理掉其他内存。

#### 分代回收

​       根据对象存活的周期，将内存分为新生代和老年代。

​       新生代：对象存活率低，每次都有大量对象死去，采用复制算法。

​       老年代：对象存活率高，没有额外的内存对它进行内存担保，所以使用标记清理或者标记整理算法。

 

#### 垃圾回收器

可达性分析对执行时间的敏感中的一致性：

​       整个可达性分析期间整个执行系统看起来就像被冻结在某个时间点上，不可以出现分析过成中对象引用关系还在不断变换的情况，该点不满足的话，分析结果准确性就无法得到保障。这点是导致GC进行时必须停顿所有Java执行线程（Stop The world）的其中一个重要原因。

​       安全点：程序执行时并非在所有的地方都能停顿下来GC，只有在到达安全点时才能暂停。安全点的选定既不能太少让GC等待时间太长，也不能过于频繁以至于过分增大运行时负荷，所以安全点的选定，基本上是以程序“是否具有让程序长时间执行的特征”为标准进行选定的。包含抢先式中断和主动式中断

​       抢先式中断：不需要线程的执行代码主动去配合，GC发生时，首先把所有的线程全部中断，如果发现有线程的中断店不在安全点上，就恢复线程，让它跑到中断点上。

​       主动式中断：当GC需要中断线程时，不直接对线程操作，仅仅简单的设置一个标志，各个线程主动去轮询这个标志，发现中断标志为真时，就自己中断挂起。

​       安全区域：如果线程处于Sleep或者Block状态，无法响应JVM的中断请求，就需要安全区域解决。指在一段代码片段之中，引用关系不会发生变化，这个区域中的任何地方开始GC都是安全的

1.     Serial收集器

单线程收集器，并不仅仅说明只会使用一个CPU或一条收集线程去完成垃圾收集工作，重要的是在它进行垃圾收集时，必须暂停其他所有的工作线程，直到它收集结束。

2.     ParNew 收集器

Serial收集器的多线程版本

3.     Parallel Scavenge 收集器

复制算法收集器，

4.     Serial Old收集器

采用标记整理算法的单线程收集器

5.      Parallel Old 收集器

多线程和标记整理算法

6.      CMS 收集器

标记清除算法

7． G1收集器

并行与并发、分代收集、标记整理

#### 内存分配与回收策略

1.     对象优先在Eden分配

大多数情况下，对象在新生代Eden中分配。当Eden区中没有足够的空间分配时，虚拟机将发起一次Minor GC，如果空间还不够，大对象直接进入老年代。

2.     长期存活的对象将进入老年代

从Eden中每经过一次GC，增加1，默认达到15时，晋升如老年代。但并不是必须要达到规定数值，如果在Survivor空间中相同年龄所有对象大小的综合大于Survivor空间的一半，大于或等于该数值的对象，直接进入老年代

####  空间分配担保

Minor GC 前，如果Survivor无法容纳存活对象，会在老年代的分配担保下，进入老年代，（参考经验值同样为：老年代最大连续空间是否大于历次晋升到老年代的平均大小），如果担保失败，那么就进行一次Full GC。

Minor GC 前，检查老年代最大可用连续空间是否大于新生代所有对象总和，

- 分配担保-允许：老年代最大连续空间是否大于历次晋升到老年代的平均大小
  - 大于：尝试Minor GC+分配担保
  - 小于：Full GC
- 分配担保-不允许：Full GC



### 类加载

#### 类加载过程

加载-验证-准备-解析-初始化-使用-卸载

1.     加载

1)     通过一个类的全限定名来获取定义此类的二进制字节流

2)     将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构

3)     在内存中生成一个代表这个类的java.lang.Class对象，作为方法区这个类的数据的访问入口

2.     验证

目的是确保Class文件的字节流中包含的信息符合当前虚拟机的要求，并且不会危害虚拟机的自身安全。

3.     准备

正式为类变量分配内存并设置类变量初始值的阶段，这些变量所使用的内存都将在方法区进行分配。这里内存分配的仅包括类变量（static变量），而不包括实例变量，实例变量将会在对象实例化时随着对象一起分配在Java堆中。

注：

例：public static int value = 123；变量在准备阶段的初始化过后的初始化为0而不是123，赋值指令是在程序编译后，存放于类构造器方法中，赋值动作将在初始化阶段才会执行。

4.     解析

将常量池中的符号引用替换为直接引用的过程。

5.     初始化

初始化类，真正开始执行类中定义的Java程序代码（字节码）。

**类初始化的时机**

1. 遇到new、getstatic、putstatic、invokestatic指令时，如果没有初始化，需先初始化
2. 使用java.lang.reflect 的方法对类进行反射调用时
3. 初始化一个类时，发现其父类还没有初始化，先初始化其父类
4. 虚拟机启动时，初始化执行主类
5. 1.7动态语言支持，句柄锁对应的类还没有初始化，先初始化

注意的细节：

1. 对于静态字段，只有直接定义这个字段的类才会被初始化。

   通过子类引用父类的静态字段，不会导致子类初始化，只会出发父类的初始化

2. 数组定义引用类，不会触发其初始化
3. 常量在编译阶段会存入常量池中，本质上并没有直接引用到定义的类，不会触发类的初始化

#### 类的回收->回收方法区

同时满足：

1.     该类所有的实例都已经被回收，Java堆中不存在该类的任何实例

2.     加载该类的ClassLoader已经被回收

3.     该类对应的java.lang.Class对象没有再任何地方被引用，无法在任何地方通过反射访问该类的方法

#### 双亲委派

工作过程：

如果一个类加载器收到了类加载请求，首先不会自己尝试加载这个类，而是把这个请求委派给父类加载器去完成，每一个层次的类加载器都是如此，因此所有的加载请求最终都应该传送到顶层的启动类加载器中，只有当父类加载器反馈无法自己完成这个加载请求时，子加载器才会尝试自己去加载。

 

 

#### 类加载器

类加载阶段中“通过一个类的全限定名来获取描述此类的二进制流”这个动作的代码模块称之为类加载器

比较两个类是否相等，只有在这两个类是由同一个类加载器加载的前提下才有意义，否则只要加载它们的类加载器不同，这两个类也就必定不相等。（包括Instanceof 关键字运算结果）

 

1.     启动类加载器Bootstrap ClassLoader

2.     扩展类加载器Extension ClassLoader

3.     应用程序类加载器Application ClassLoader，也被成为系统加载器

###   创建对象的过程

1. 检查指令对应的参数是否能在常量池中定位到这个类的符号引用，并检查是否已被加载、解析和初始化。

2. 为新生对象分配内存，所需内存大小在加载完成后已经确定

   > 内存分配：
   >
   > 1. 指针碰撞：移动指针分配内存空间
   >
   > 2. 空闲列表：记录可用内存地址，从空闲列表中分配
   >
   > 内存分配线程安全问题：
   >
   > 1. 同步处理：CAS 失败重试 保证原子性
   > 2. 本地线程缓冲区：在线程的缓冲区分配

3. 初始化零值

4. 设置对象信息：对应的类、数据信息、分代年龄等，这些数据存放在对象头中

   > 模拟机角度来看，已经完成对象创建，
   >
   > java角度看，对象创建才刚刚开始

5. 执行对应的<init> 方法

**静态变量初始化**

- 准备阶段初始化为默认值，如int 则为0
- 赋值动作在jvm中类初始化阶段执行

### 对象访问的定位

1. 句柄 

   优点：对象移动只需要更改句柄中的指针

2. 直接指针

   优点：速度更快，节省了一次指针定位



### 局部变量问题

局部变量表存放于栈->栈帧中，容量以变量槽（Slot）为最小单位，为了节省栈帧空间，Slot是可复用的，在一些情况下，会影响内存回收问题。

```java
//1.
public static void main(){
	byte[] a = new byte[1024*1024];
    //a并没有被回收，还处于变量作用域中。
	System.gc();
}
//2.
public static void main(){
    {
        byte[] a = new byte[1024*1024];
    }
    //a还是没有被回收。
    //原因：
    //没有任何局部变量表的读写操作，变量a占用的Slot还没有被其他变量所复用，作为GC Roots一部分的局部变量表仍然保持着对该变量的引用
	System.gc();
}

//3.
public static void main(){
    {
        byte[] a = new byte[1024*1024];
    }
    int b = 0；
    //a被回收。
    //Slot 被复用，引用消除
	System.gc();
}
```

**总结**：

局部变量可能不回收，可能是因为局部变量表中的Slot没有被复用，作为GC Roots的一部分，还保持引用的原因。

**解决**：

- **存在问题的手动赋值null**
  - 可行的地方：特殊情况下（对象占有内存较大、此方法栈帧长时间不能被回收、方法调用次数达不到JIT编译条件）的情况下
  - 问题：赋值为null的操作在经过JIT编译优化后，就会被移除
- 以恰当的变量作用域来控制变量回收才是最优雅的解决办法



### 解释器与编译器

- 解释器：程序需要迅速启动和执行的时候，省去编译时间，立即执行

- 编译器：程序运行后，把越来越多的代码编译成本地代码，获取更高的执行效率

  编译对象与触发条件：

  1. 被多次调用的方法
  2. 被多次执行的循环体

#### JIT

运行期编译器（动态编译），把字节码转换成机器码的过程

优点：

-  可以根据当前程序的运行情况生成最优的机器指令序列
-  当程序需要支持动态链接时，只能使用JIT
- 可以根据进程中内存的实际情况调整代码，使内存能够更充分的利用

缺点：

- 编译需要占用运行时资源，会导致进程卡顿
- 由于编译时间需要占用运行时间，对于某些代码的编译优化不能完全支持，需要在程序流畅和编译时间之间做权衡
- 在编译准备和识别频繁使用的方法需要占用时间，使得初始编译不能达到最高性能

#### AOT

提前编译器（静态编译）：直接把java文件编译成机器码的过程

优点：

- 在程序运行前编译，可以避免在运行时的编译性能消耗和内存消耗
- 可以在程序运行初期就达到最高性能
- 可以显著的加快程序的启动

缺点：

- 在程序运行前编译会使程序安装的时间增加
- 牺牲Java的一致性
- 将提前编译的内容保存会占用更多的外



##  Java多线程

### 线程

​	线程是进程中可独立执行的最小单位，也是CPU分配的基本单位，每个线程有自己的程序计数器和栈区域。



#### 线程的实现方式

 1. 实现Runnable接口

 2. 继承Thread类

    

​        调用Start后，线程并没有马上执行，而是处于就绪状态，等待CPU获取资源后真正运行，run方法执行完成后，线程就处于终止状态。

#### 线程的状态

1. 新建（NEW）
2. 可运行(RUNNABLE)：就绪
3.  运行(RUNNING)：运行中
4. 阻塞(BLOCKED)：
   - 等待阻塞：如调用wait，被放入等待队列时
   - 同步阻塞：获取对象同步锁时的阻塞
   - 其他阻塞：如sleep
5. 死亡(DEAD)：线程执行结束，线程run()、main() 方法执行结束，或者因异常退出了run()方法

#### 线程通知与等待

 1. wait()函数

    调用线程会被阻塞挂起，直到一下情况之一：

    1)  其他线程调用了该共享对象的notify()或者notifyAll()方法

    2)  其他线程调用了该线程的interrupt方法，该线程抛出InterruptException异常返回

    

    需要注意：

    1)  如果调用wait()方法的线程没有实现获取该对象的监视器锁，则调用wait()方法时，调用线程会抛出IllegalMonitorStateException异常。

    > 那么 个线程如何才能获取一个共享变量的监视器锁呢
    >
    > ( ）执行 synchronized 步代码块时 使用该共享变量作为参数。
    >
    > synchronzed （共享变量）｛
    >
    > ​	//doSomething 
    >
    > }
    >
    > (2 ）调用该共享变量的方法，并且该方法使用了 synchronized 修饰
    >
    > synchronized void add (int a , int b) {
    >
    > ​	//doSomething 
    >
    > ｝

    2)  一个线程可以从挂起状态变为运行状态，即时该线程没有被其他线程调用norify()、notifyAll()方法进行通知，或者被中断，或者等待超时，这就是所谓的虚假唤醒。

    例：（生产者消费者）

    ```java
    //生产线程
    synchronized(queue){
    	while(queue.size()==MAX_SIZE){
    		try{
    			／／挂起当前线程， 并释放通过同步块获取的queue上的锁，，上消费者线程可以获取该锁，然后
    获取队列里面的元素
    			queue.wait();
    		}catch(Exception e){
    			e.printStackTrace();
    		}
    		／／ 空闲则生成元素， 并通知消费者线程
    		queue.add(ele) ;
    		queue.notifyAll();
    	}
    }
    // 消费者线程
    synchronized(queue){
    	while(queue.size()==0){
    		try{
    			／／挂起当前线程，并释放通过同步块获取的queue 上的锁，让生产者线程可以获取该锁，将生产元素放入队列
    			queue.wait();
    		}catch(Exception e){
    			e.printStackTrace();
    		}
    		／／ 空闲则生成元素， 并通知消费者线程
    		queue.take() ;
    		queue.notifyAll();
    	}
    }
    ```

    注：

    当前线程调用共享变量的wait()方法后只会释放当前共享变量上的锁，如果当前线程还持有其他共享变量的锁，则这些锁是不会释放的。

2. wait(ling timeout)函数

   如果一个线程调用共享对象的该方法挂起后，没有在指定的timeout ms时间内被其他线程调用该变量的notify()、notifyAll()唤醒，那么会因为超时而返回。wait() 其实就是wait(0)。如果传递了一个负的timeout则会抛出IllegalArgumentException异常。

3. wait(long timeout, int nanos)

4. notify () 函数

   一个线程调用共享对象的notify（）方法后，会唤醒一个在该共享变量上调用wait 系列方法后被挂起的线程。一个共享变量上可能会有多个线程在等待，具体唤醒哪个等待的线程是随机的。只有该线程竞争到了共享变量的监视器锁后，才可以继续执行。

5. notifyAll() 函数

   不同于在共享变量上调用notify（）函数会唤醒被阻塞到该共享变量上的一个线程，notifyAll() 方法则会唤醒所有在该共享变量上由于调用wait 系列方法而被挂起的线程。

   在共享变量上调用notifyAll() 方法只会唤醒调用这个方法前调用了wait 系列函数而被放入共享变量等待集合里面的线程。如果调用notifyAll()  方法后一个线程调用了该共享变量的wait() 方法而被放入阻塞集合， 则该线程是不会被唤醒的。

   

#### 等待线程执行终止的join方法

```java
threadOne . join () ;
threadTwo . join () ;
```

首先会在调用threadOne.join()方法后被阻塞， 等待threadOne 执行完毕后返回。threadOne 执行完毕后threadOne.join() 就会返回， 然后主线程调用threadTwo.join（） 方法后再次被阻塞， 等待threadTwo 执行完毕后返回。

线程A 调用线程B 的join 方法后会被阻塞， 当其他线程调用了线程A 的interrupt（）方法中断了线程A 时，线程A 会抛出InterruptedException 异常而返回。



#### 睡眠的sleep方法

Thread 类中有一个静态的sleep 方法，当一个执行中的线程调用了Thread 的sleep 方法后，调用线程会暂时让出指定时间的执行权，也就是在这期间不参与CPU 的调度，但是该线程所拥有的监视器资源，比如**锁还是持有不让出的**。

如果在睡眠期间其他线程调用了该线程的interrupt()方法中断了该线程，则该线程会在调用sleep 方法的地方抛出IntermptedException 异常而返回。



#### wait()与sleep()的区别

**简单来说wait()会释放对象锁资源而sleep()不会释放对象锁资源。但是 wait 和sleep 都会释放cpu资源** 



#### 让出CPU 执行权的yield 方法

Thread 类中有一个静态的yield 方法，当一个线程调用yield 方法时，实际就是在暗示线程调度器当前线程请求让出自己的CPU 使用，但是**线程调度器可以无条件忽略这个暗示**。

***sleep 与yield 方法的区别在于***，*当线程调用sleep 方法时调用线程会被阻塞挂起指定的时间，在这期间线程调度器不会去调度该线程。而调用yield 方法时，线程只是让出自己剩余的时间片，并没有被阻塞挂起，而是处于就绪状态，线程调度器下一次调度时就有可能调度到当前线程执行。*



#### 线程中断

Java 中的线程中断是一种线程间的协作模式，通过设置线程的中断标志并不能直接终止该线程的执行， 而是被中断的线程根据中断状态自行处理。

-  void interrupt() 方法： 中断线程

-  boolean isInterrupted()  方法： 检测当前线程是否被中断
- boolean interrupted()  方法： 检测当前线程是否被中断， 如果是返回true ， 否则返回false 。与is lnterrupted 不同的是，该方法如果发现当前线程被中断， 则会清除中断标志，并且该方法是static 方法， 可以通过Thread 类直接调用。



#### 线程死锁

死锁的产生必须具备以下四个条件：

- **互斥条件**： 指线程对己经获取到的资源进行排它性使用， 即该资源同时只由一个线程占用。如果此时还有其他线程请求获取该资源，则请求者只能等待，直至占有资源的线程释放该资源。
- **请求并持有条件**： 指一个线程己经持有了至少一个资源， 但又提出了新的资源请求，而新资源己被其他线程占有，所以当前线程会被阻塞，但阻塞的同时并不释放自己己经获取的资源。
- **不可剥夺条件**： 指线程获取到的资源在自己使用完之前不能被其他线程抢占， 只有在自己使用完毕后才由自己释放该资源。
- **环路等待条件**： 指在发生死锁时， 必然存在一个线程→资源的环形链， 即线程集合{TO , TL T2 ，…， Tn ｝中的TO 正在等待一个Tl 占用的资源， Tl 正在等待T2 占用的资源，……Tn 正在等待己被TO 占用的资源。

```java
ThreadA:

public void run(){
	synchronized ( resourceA) {
		Thread.sleep (l000 );
		synchronized ( resourceB) {
			// doSomething
		}
	}
}

ThreadB:

public void run(){
	synchronized ( resourceB) {
		Thread.sleep (l000);
		synchronized (resourceA) {
			// doSomething
		}
	}
}
```

以上伪代码案例构成了环路等待的条件，所以进入了死锁。上述案例的避免死锁方法，顺序请求资源的锁，即ThreadB中请求资源锁的顺序，与ThreadA 中的请求顺序相同即可。



#### 守护线程与用户线程

Java 中的线程分为两类，分别为daemon 线程（守护线程〉和user 线程（用户线程）。在JVM 启动时会调用main 函数， main 函数所在的钱程就是一个用户线程。JVM中只要有一个用户线程还没结束， 正常情况下NM 就不会退出。

如果你希望在主线程结束后JVM 进程马上结束，那么在创建线程时可以将其设置为守护线程，如果你希望在主线程结束后子线程继续工作，等子线程结束后再让JVM 进程结束，那么就将子线程设置为用户线程。

#### ThreadLocal

提供了线程本地变量，多个线程同时操作同一个变量时，实际操作的是自己本地内存里面的变量，从而避免了线程安全问题。创建一个ThreadLocal 变量后，每个线程都会复制一个变量到自己的本地内存。

实现原理：

​		ThreadLocalMap 是一个定制化的Hashmap 。ThreadLocal 类型的本地变量存放在具体的线程内存空间中。ThreadLocal 就是一个工具壳，它通过set 方法把value 值放入调用线程的threadLocals 里面并存放起来， 当调用线程调用它的get 方法时，再从当前线程的threadLocals 变量里面将其拿出来使用。如果调用线程一直不终止， 那么这个本地变量会一直存放在调用线程的threadLocals 变量里面，所以当不需要使用本地变量时可以通过调用ThreadLocal 变量的remove 方法，从当前线程的threadLocals 里面删除该本地变量。

**同一个ThreadLocal 变量在父线程中被设置值后， 在子线程中是获取不到的。**



#### InheritableThreadLocal

**让子线程可以访问在父线程中设置的本地变量**。解决同一个ThreadLocal 变量在父线程中被设置值后， 在子线程中是获取不到的问题。

线程在通过InheritableThreadLocal 类实例的set 或者get 方法设置变量时，就会创建当前线程的inheritableThreadLocals 变量。当父线程创建子线程时，构造函数会把父线程中inheritableThreadLocals 变量里面的本地变量复制一份保存到子线程的inheritableThreadLocals 变量里面。



### synchronized

加锁与释放锁的语义：

- 进入synchronized块的内存语义是把在synchronized块内使用到的变量从线程的工作内存中清除，这样在synchronized块内使用到该变量时就不会从线程的工作内存中获取

- 而是直接从主内存中获取。退出synchronized块的内存语义是把在synchronized块内对共享变量的修改刷新到主内存。



### volatile

当线程写入了volatile变量值时就等价于线程退出volatile同步块（把写入工作内存的变量值同步到主内存），读取volatile变量值时就相当于进入同步块（ 先清空本地内存变量值，再从主内存获取最新值） 。

1. 保证内存可见性
2. 避免指令重排

**使用volatile关键字的场景**

- **写入变量值不依赖、变量的当前值时**。因为如果依赖当前值，将是获取一计算一写入三步操作，这三步操作不是原子性的，而volatile不保证原子性。
- **读写变量值时没有加锁**。因为加锁本身已经保证了内存可见性，这时候不需要把变量声明为volatile的。



### CAS

volatile 只能保证共享变量的可见性，不能解决读改一写等的原子性问题。CAS 即Compare and Swap ，其是JDK 提供的非阻塞原子性操作， 它通过硬件保证了比较更新操作的原子性。

ABA问题：

1. 线程A 操作value=1
2. 线程B 操作value=2
3. 线程A 操作value=1

​	变量的状态发生了环形转换。

​	JDK 中的AtomicStampedReference 类给每个变量的状态值都配备了一个时间戳， 从而避免了ABA 问题的产生。



### Unsafe类

JDK 的此jar 包中的Unsafe 类提供了硬件级别的原子性操作， Unsafe 类中的方法都是native 方法，它们使用刑I 的方式访问本地C＋＋实现库。

### Atomic相关

AtomicLong、AtomicInteger、AtomicBoolean

基于Unsafe 用CAS实现的原子操作类，AtomicLong：原子性递增或递减

### LongAdder

LongAdder是LongAccumulator中的特例。

AtomicLong基础上的实现，内部维护多个变量，供多线程去请求竞争锁，获取值时，把所有变量的值累加再加上base返回。最大的变量数默认是CPU核数

扩容：只有核数小于CPU核数，且当前多线程访问了同一个元素，导致其中一个线程CAS失败冲突时，才会进行扩容操作。

LongAccumulator 与 LongAdder区别

| LongAccumulator     | LongAdder         |
| ------------------- | ----------------- |
| 可以提供非0的初始值 | 只能提供默认的0值 |
| 可以指定累加规则    | 内置累加规则      |



### AQS-CAS

抽象同步队列，，实现同步器的基础组件，并发包中的锁底层就是使用AQS实现的。是一个FIFO的双向队列

- AQS 是一个FIFO 的双向队列，其内部通过节点head 和tail 记录队首和队尾元素，队列元素的类型为Node 。
- 其中Node中的thread 变量用来存放进入AQS队列里面的线程；
- Node 节点内部的SHARED 用来标记该线程是获取共享资源时被阻塞挂起后放入AQS 队列的；
- EXCLUSIVE 用来标记线程是获取独占资源时被挂起后放入AQS 队列的
- waitStatus 记录当前线程等待状态，可以为CANCELLED （线程被取消了）、SIGNAL （ 线程需要被唤醒）、C ONDITION （线程在条件队列里面等待〉、PROPAGATE （释放共享资源时需要通知其他节点〕；

在AQS 中维持了一个单一的状态信息state，可以通过getState 、setState 、compareAndSetState 函数修改其值。

- 对于ReentrantLock来说state 可以用来表示当前线程获取锁的可重入次数；
- 对于读写锁ReentrantReadWriteLock 来说，state 的高16位表示读状态，也就是获取该读锁的次数，低16 位表示获取到写锁的线程的可重入次数；
- 对于CountDownlatch 来说，state 用来表示计数器当前的值
- 对于Semaphore来说，state 用来表示当前可用信号的个数

对于AQS 来说，线程同步的关键是对状态值state 进行操作。根据state 是否属于一个
线程，操作state 的方式分为独占方式和共享方式。

#### 原理

使用独占方式获取的资源是与具体线程绑定的，就是说如

- 独占方式 ：使用独占方式获取的资源是与具体线程绑定的
  1. 如当一个线程获取了ReetrantLock 的锁后，
  2. 在AQS 内部会首先使用**CAS 操作**把state 状态值从0变为1 ，然后设置当前锁的持有者为当前线程
  3. 当该线程再次获取锁时发现它就是锁的持有者，则会把状态值从l 变为2 ，也就是设置可重入次数
  4. 当另外一个线程获取锁时发现自己并不是该锁的持有者就会被放入AQS 阻塞队列后挂起。
- 共享方式：资源与具体线程是不相关的
  1. 当一个线程获取到了资源后
  2. 另外一个线程再次去获取时如果当前资源，如Semaphore信号量。
  3. 如果还能满足它的需要，则通过自旋CAS 获取信号量。
  4. 不满足则把当前线程放入阻塞队列，

#### AQS--条件变量的支持

与notify 和wait是配合synchronized 内置锁实现线程间同步的基础设施一样，条件变量的signal 和await 方法也是用来配合锁（使用AQS 实现的锁〉实现线程间同步的基础设施。

**（notify 和wait）与（signal 和await ）不同点：** 

​	synchronized 同时只能与一个共享变量的notify 或wait 方法实现同步，而AQS 的一个锁可以对应多个条件变量。

在调用共享变量的notify 和wait 方法前**必须先获取该共享变量的内置锁**，同理，在调用条件变量的signal 和await 方法前也**必须先获取条件变量对应的锁**。

```java
// (l) 基于AQS实现的独占锁ReentranLock对象
ReentrantLock lock= new ReentrantLock() ; 
//(2)ConditionObject对象，变量就是Lock 锁对应的－个条件变量，
//注意：一个对象可创建多个条件变量！
Condition condition = lock.newCondition(); 
// (3) 获取独占锁
lock.lock() ; 
try {
	System.out.println("begain wait ") ;
     //(4) 条件变量阻塞挂起当前线程
    // 当其他线程调用条件变量的signal方法时，被阻塞的线程才会从await处返回
    // 如果此时没有获取到锁，回抛出java.lang.IllegalMonitorStateException 异常。
	condition.await() ;
	System.out.println("end wait");
} catch (Exception e) {
	e.printStackTrace () ;
} finally {
     //(5)释放对象锁    
    lock.unlock() ;
}

lock. lock() ; // (6)
try {
System.out.println("begain wait") ;
	condition. await() ; //( 7)
	System.out.println("end wait");
} catch (Exception e) {
	e.printStackTrace () ;
} finally {
    // (8)释放对象锁    
	lock.unlock() ; 
}
```



### 伪共享

伪共享：当多个线程同时修改一个缓存行里面的多个变量时，由于同时只能有一个线程操作缓存行，所以相比将每个变量放到一个缓存行，性能会有所下降。

伪共享的产生是因为多个变量被放入了一个缓存行中，并且多个线程同时去写入缓存行中不同的变量。

避免伪共享：JDK 8 提供了一个sun.misc.Contended 注解，用来解决伪共享问题。



### 锁的分类、概念

- 乐观锁：读取直到更改前都不加锁，更改时才加锁。
- 悲观锁：先加锁，再读取、修改，最后释放锁
- 公平锁：线程获取锁的顺序是按照线程请求锁的时间早晚来决定
- 非公平锁：运行时闯入，随机获取锁
- 可重入锁：当一个线程再次获取已经获取的锁的资源，不被阻塞，即为可重入锁
- 自旋锁：当前线程在获取锁时，如果发现锁已经被其他线程占有，不马上阻塞自己，在不放弃CPU使用权的情况下多次尝试获取

### 原子变量操作类

JUC 并发包中包含有Atomiclnteger 、AtomicLong 和AtomicBoolean 等原子性操作类，这些操作类在rt.jar 包下面的，并且是通过BootStarp 类加载器进行加载的，所以能通过Unsafe.getUnsafe ( ）方法获取到Unsafe 类的实例，使用CAS实现原子性操作。



### 独占锁ReentrantLock-AQS

ReentrantLock 是可重入的独占锁，同时只能有一个线程可以获取该锁，其他获取该锁的线程会被阻塞放入该锁的AQS阻塞队列中。

案例：独占锁实现简单的线程安全List

```java
public static class ReentrantLockList {
／／线程不安全的list
private ArrayList<String> array ＝ new ArrayList<String> () ;
／／独占锁
private volatile ReentrantLock lock= new ReentrantLock() ;

	private void add(String e){
		lock.lock();
		try{
			array.add(e);
		}finally{
			lock.unlock();
		}
	}

	private void get(int index){
		lock.lock();
		try{
			array.get(index);
		}finally{
			lock.unlock();
		}
	}
	
	private void remove(String e){
		lock.lock();
		try{
			array.remove(e);
		}finally{
			lock.unlock();
		}
	}
}
```

ReentrantLock 的底层是使用AQS 实现的可重入独占锁。在这里AQS 状态值为0 表示当前锁空闲，为大于等于1的值则说明该锁己经被占用。该锁内部有公平与非公平实现， 默认情况下是非公平的实现



### CountDownLatch-AQS

构造方法中的参数，是指线程个数，可以使用CountDownLatch#countDown() 递减，多个线程执行时，可使用CountDownLatch计数，如任务3基于任务2与任务1完成后执行，CountDownLatch(2)，在任务1与任务2的完成后调用CountDownLatch#countDown() 递减其计数。

CountDownLatch 的方法不是很多，将它们一个个列举出来：

1. await() throws InterruptedException：调用该方法的线程等到构造方法传入的 N 减到 0 的时候，才能继续往下执行；
2. await(long timeout, TimeUnit unit)：与上面的 await 方法功能一致，只不过这里有了时间限制，调用该方法的线程等到指定的 timeout 时间后，不管 N 是否减至为 0，都会继续往下执行；
3. countDown()：使 CountDownLatch 初始值 N 减 1；
4. long getCount()：获取当前 CountDownLatch 维护的值；

```java
CountDownLatch startSignal = new CountDownLatch(2);
Thread1{
    try{
        Thread.sleep(1000);
    }catch(Exception e){
    }finally{
        startSignal.countDown();
    }
}
Thread2{
    try{
        Thread.sleep(1000);
    }catch(Exception e){
    }finally{
        startSignal.countDown();
    }
}
Thread1.start();
Thread1.start();
// 阻塞，直到当其中的信号量为0时，
startSignal.await();
```

#### 原理

底层是基于AQS实现的，实际上是吧计数器的值赋值给了AQS的状态变量state。

#### CountDownLatch与Join的区别

| CountDownLatch                                               | Join                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 允许子线程在运行完毕或者在运行中递减计数，(即，CountDownLatch可以在子线程运行的任何时刻让await方法返回而不一定必须要等到线程结束) | 调用一个子线程的join方法后，该线程会一直被阻塞到子线程执行完毕 |
| 使用线程池时，更灵活                                         | 线程池中不好实现                                             |



### 回环屏障CyclicBarrier-AQS

让一组线程全部达到一个状态后再全部执行。线程调用await方法后就会被阻塞（屏障点），等所有线程都调用了await方法后，所有线程都会冲破屏障继续执行。**基于独占锁实现，底层还是基于AQS**

```java
private static CyclicBarrier cyclicBarrier= new CyclicBarrier (2, new Runnable()
{
	public void run(){
		// dosomething 所有子线程全部到达屏障后执行的任务
	}
}
                                                               
//将线程A添加到线程池
// executorService 线程池
executorService.submit(new Runnable () {
public void run() {
try {
//dosomething
cyclicBarrier.await ();
//dosomething
} catch (Exception e) {
e . printStackTrace() ;
} ) ;
    ...
```

#### CyclicBarrier与CountDownLatch的区别

- CyclicBarrier可复用；count计数器变为0后，会用初始化的值给count再次赋值

- CountDownLatch是一次性的

### 信号量Semaphore-AQS

java中的一个同步器，计数器是递增的，初始化时候指定一个初始值。与CyclicBarrier不同的是计数器不会自动重置

- release()：count++
- release(int permits)：count+=permits
- acquire() ：希望获取一个信号量资源，如果大于0，计数器-1，方法直接返回；如果等于0，当前线程放入阻塞队列
- acquire(int permits)：

```java
／／创建一个Semaphore 实例,初始值为0
private static Semaphore semaphore = new Semaphore(O) ;
// 相当于count++，当前线程阻塞放入AQS阻塞队列，等待激活
semaphore.release( );
// 阻塞到count==2，唤醒阻塞队列
semaphore.acquire (2) ;
```



### 线程安全List - CopyOnWriteArrayList

并发包中的并发List 只有CopyOnWriteArrayList 。CopyOnWriteArray List 是一个线程安全的ArrayList ，对其进行的修改操作都是在底层的一个复制的数组（快照）上进行的，也就是使用了写时复制策略。其中写时使用的是**ReentrantLock 独占锁**

**add**：

	1. 获取独占锁

   	2. 复制array到一个新数组，并把新增的元素添加到新数组
            	3. 使用新数组替换原数组，并在返回前释放锁

**get**：

​	获取array数组，通过下标访问指定位置元素。

​	get中没有加锁，所以可能会导致写时复制策略产生的弱一致问题。

**set**：

	1. 获取独占锁

   	2. get指定位置元素
            	3. 指定位置元素值与新值不一致，则创建新数组并复制元素，在新数组上修改指定位置的元素，并设置新数组到array。
         	4. 指定位置元素值与新值一致，为了保证volatile语义，还是会重新设置array

**remove**：

	1. 获取独占锁

   	2. 将除了要移除的元素以外的其他数组复制到新数组
            	3. 将新数组设置到array



​		CopyOnWriteArrayList 使用写时复制的策略来保证list 的一致性，而获取一修改一写入三步操作并不是原子性的，所以在增删改的过程中都使用了独占锁，来保证在某个时间只有一个线程能对list 数组进行修改。另外CopyOnWriteArrayList 提供了弱一致性的法代器， 从而保证在获取迭代器后，其他线程对list 的修改是不可见的， 迭代器遍历的数组是一个快照。



### 读写锁ReentrantReadWritelock-AQS

AQS state 高16位表示读锁次数，低16位表示写锁可重入次数。

写锁是个独占锁， 某时只有一个线程可以获取该锁。

- 如果当前没有线程获取到读锁和写锁， 则当前线程可以获取到写锁然后返回。
- 如果当前己经有线程获取到读锁和写锁，则当前请求写锁的线程会被阻塞挂起。

获取读锁

- 如果当前没有其他线程持有写锁，则当前线程可以获取读锁
- 如果其他一个线程持有写锁， 则当前线程会被阻塞

案例：读写锁实现线程安全List

```java
public static class ReentrantLockL 工st {
／／线程不安全的list
private ArrayList<String> array = new ArrayList<String> ();
独占锁
private final ReentrantReadWriteLock lock ＝new ReentrantReadWriteLock() ;
private final Lock readLock = lock.readLock () ;
private final Lock writeLock = lock.writeLock ();

／／添加元素
public void add (String e) {
	writeLock.lock ();
	try {
		array.add (e ) ;
	} finally {
		writeLock.unlock() ;
	}
}
    
／／删除元素
public void remove (String e) {
	writeLock.lock ();
	try {
		array.remove (e ) ;
	} finally {
		writeLock.unlock() ;
	}
}
    
／／获取数据
public void get (int index ) {
	readLock.lock ();
	try {
		array.get (index ) ;
	} finally {
		readLock.unlock() ;
	}
}
```

### 并发队列

#### ConcurrentlinkedQueue

线程安全的无界非阻塞队列，其底层数据结构使用单向链表实现，对于入队和出队操作使用CAS 来实现线程安全。

#### LinkedBlockingQueue

独占锁实现的阻塞队列，默认队列容量为Ox7fffffff，用户也可以自己指定容量，所以从一定程度上可以说LinkedBlockingQueue 是有界阻塞队列。

#### ArrayBlockingQueue

有界数组方式实现的阻塞队列，使用全局独占锁实现了同时只能有一个线程进行入队或者出队操作

#### PriorityBlockingQueue

带优先级的无界阻塞队列，每次出队都返回优先级最高或者最低的元素。其内部是使用平衡二叉树堆实现的，所以直接遍历队列元素不保证有序。默认使用对象的compareTo 方法提供比较规则，如果你需要自定义比较规则则可以自定义comparators 。

#### DelayQueue

并发队列是一个无界阻塞延迟队列，队列中的每个元素都有个过期时间，当从队列获取元素时，只有过期元素才会出队列。队列头元素是最快要过期的元素。其内部使用PriorityQueue 存放数据，使用ReentrantLock 实现线程同步。



### 线程池ThreadPoolExecutor

ThreadPoolExecutor 的实现实际是一个生产消费模型。

- **newFixedThreadPool** ：创建一个核心线程个数和最大线程个数都为nThreads 的线程池，并且阻塞队列长度为Integer.MAX_VALUE。keeyAliveTime = 0 说明只要线程个数比核心线程个数多并且当前空闲则回收

  ```java
  public static ExecutorService newFixedThreadPool （int nThreads) {
  	return new ThreadPoolExecutor(nThreads , nThreads ,
  						0L , TimeUnit.MILLISECONDS ,
  						new LinkedBlockingQueue<Runnable>()) ;
  }
  ```

  

- **newSingleThreadExecutor** ： 创建一个核心线程个数和最大线程个数都为1 的线程池，并且阻塞队列长度为Integer.MAX_VALUE。keeyAliveTime = 0说明只要线程个数比核心线程个数多并且当前空闲则回收。

  ```java
  public static ExecutorService newSingleThreadExecutor() {
  return new FinalizableDelegatedExecutorService
  			( new ThreadPoolExecutor(l , 1 ,
  									OL, TimeUnit.MILLISECONDS ,
  									new LinkedBlockingQueue<Runnable>())) ;
  ```

- **newCachedThreadPool** ： 创建一个按需创建线程的线程池，初始线程个数为0 ， 最多线程个数为Integer. MAX_VA LUE ，并且**阻塞队列为同步队列**。keeyAliveTime=60说明只要当前线程在60s内空闲则回收。这个类型的特殊之处在于， **加入同步队列的任务会被马上执行，同步队列里面最多只有一个任务**。

  ```
  public static ExecutorService newCachedThreadPool () {
  	return new ThreadPoolExecutor(O , Integer .MAX_VALUE ,
  								60L ,  TimeUnit.MILLISECONDS ,
  								new SynchronousQueue<Runnable> ());
  }
  ```

  

**shutdown 操作**:

​		调用shutdown 方法后，线程池就不会再接受新的任务了，但是工作队列里面的任务还是要执行的。该方法会立刻返回，并不等待队列任务完成再返回。

**shutdownNow 操作**：

​		调用shutdownNow 方法后， 线程池就不会再接受新的任务了，并且会丢弃工作队列里面的任务， 正在执行的任务会被中断， 该方法会立刻返回，并不等待激活的任务执行完成。返回值为这时候队列里面被丢弃的任务列表。

**awaitTermination 操作**:

​		当线程调用awaitTermination 方法后，当前线程会被阻塞，直到线程池状态变为TERMINATED 才返回， 或者等待时间超时才返回。



### ScheduledThreadPoolExecutor

继承自ThreadPoolExecutor，并实现了ScheduledExecutorService接口。线程池队列是DelayedWorkQueue（与DelayQueue类似的延迟队列）。

内部有一个变量period表示任务类型：

- period=0 ： 一次性任务，执行完就退出
- period<0：固定延迟的定时可重复任务
- period>0：固定评论的定时可重复任务

注意：如果当前任务还没有执行完，下一次要执行的任务时间到了，并不会并发执行，下次要执行的任务会延迟执行，即等当前任务执行完毕后执行。

### ConcurrentHashMap 使用注意事项

ConcurrentHashMap 虽然为并发安全的组件，但是使用不当仍然会导致程序错误。

- put：存在则覆盖，返回原来的值；不存在则新增，返回null

- putlfAbsent：存在，返回原来的值，不覆盖；不存在，新增，返回null；**原子性操作**

所以应该**使用ConcurrentHashMap.putlfAbsent 替换 Map.put()**



### 使用ThreadLocal 不当可能会导致内存泄漏

###### 1.弱引用：

只具有弱引用的对象拥有短暂的生命周期。在垃圾回收器线程扫描它所管辖的内存区域的过程中，一旦发现了只具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存。不过，由于垃圾回收器是一个优先级很低的线程，因此不一定会很快发现那些只具有弱引用的对象。

###### 2.为什么用弱引用：

假如每个key都强引用指向ThreadLocal的对象，也就是上图虚线那里是个强引用，那么这个ThreadLocal对象就会因为和Entry对象存在强引用关联而无法被GC回收，造成内存泄漏，除非线程结束后，线程被回收了，map也跟着回收。

> 线程在可复用的情况下（例如线程池），当线程的处理任务逻辑方法中有 new ThreadLocal() 操作时会在堆中分配一块 ThreadLocal 的内存，栈中保存着其引用（图中的 ThreadLocalRef），这里肯定是强引用。当方法执行结束 ThreadLocalRef 出栈，那么堆中的 ThreadLocal 还有一条引用指向它（图中 Entry.key，也就是你纠结的为啥要声明为弱引用的引用）。线程在复用，也就是说可以通过 currentThreadRef -> currentThread -> threadLocalMap -> entry -> entry.key -> 内存中的 threadLocal。如果是强引用是不是 threadLocal 就无法被 GC 回收？但是弱引用的话，GC 就能将堆中这块内存回收，因为没有其他强引用指向它，只有这一条引用指向它。

###### 3.依然存在的内存泄漏问题：

当把ThreadLocal对象的引用置为null后，没有任何强引用指向内存中的ThreadLocal实例，threadLocals的key是它的弱引用，故它将会被GC回收。
但线程中threadLocals里的value却没有被回收，因为存在着一条从当前线程对象连接过来的强引用，且因为无法再通过ThreadLocal对象的get方法获取到这个value，它永远不会被访问到了，所以还存在内存泄漏问题。
同样的，只有在当前线程结束后，线程对象的引用不再存在于栈中，强引用断开，内存中的Current Thread、ThreadLocalMap、value才会全部被GC回收。

###### 4.解决方案：

当线程的某个ThreadLocal对象使用完了，马上调用remove方法，删除Entry对象。
其实只要这个线程对象及时被GC回收，这个内存泄露问题影响不大，只发生在ThreadLocal对象的引用设为null到线程结束的这段时间内。
但在使用线程池的时候，线程结束是不会被销毁的，会再次使用，就可能出现真正的内存泄露。

补充：
Java为了最小化内存泄露的可能性和影响，在ThreadLocal的get、set的时候，都会检查当前key所指的对象是否为null，是则删除对应的value，让它能被GC回收。



## JAVA基础相关

 

### 反射

```java
try {
    Class cls = Class.forName("com.jasonwu.Test");
    //获取构造方法
    Constructor[] publicConstructors = cls.getConstructors();
    //获取全部构造方法
    Constructor[] declaredConstructors = cls.getDeclaredConstructors();
    //获取公开方法
    Method[] methods = cls.getMethods();
    //获取全部方法
    Method[] declaredMethods = cls.getDeclaredMethods();
    //获取公开属性
    Field[] publicFields = cls.getFields();
    //获取全部属性
    Field[] declaredFields = cls.getDeclaredFields();
    Object clsObject = cls.newInstance();
    Method method = cls.getDeclaredMethod("getModule1Functionality");
    Object object = method.invoke(null);
} catch (ClassNotFoundException e) {
    e.printStackTrace();
} catch (IllegalAccessException e) {
    e.printStackTrace();
} catch (InstantiationException e) {
    e.printStackTrace();
} catch (NoSuchMethodException e) {
    e.printStackTrace();
} catch (InvocationTargetException e) {
    e.printStackTrace();
}
```

### 流

#### 字符流

**处理的最基本的单元是Unicode码元**（大小2字节），它通常用来处理文本数据。

从InputStream和OutputStream派生出来的一系列类。这类流以**字节(byte)**为基本处理单位。

◇ InputStream、OutputStream

　　◇ FileInputStream、FileOutputStream（文件流）

　　◇ PipedInputStream、PipedOutputStream

　　◇ ByteArrayInputStream、ByteArrayOutputStream

　　◇ FilterInputStream、FilterOutputStream

　　◇ DataInputStream、DataOutputStream （原始型数据流，

​                      他们是在普通流上加了读写原始型数据的功能，所以构造他们时要先构造普通流方法：

​                       readBoolean()/writeBoolean()

​                       readByte()/writeByte()

​                       readChar()/writeByte()    ）

　　◇ BufferedInputStream、BufferedOutputStream（缓冲区流）

> 对于输出地缓冲流，写出的数据，会先写入到内存中，再使用flush方法将内存中的数据刷到硬盘。所以，在使用字符缓冲流的时候，一定要先flush，然后再close，避免数据丢失。

#### 字节流

**处理的最基本单位为单个字节**，它通常用来处理二进制数据。

从Reader和Writer派生出的一系列类，这类流以**16位的Unicode码表示的字符**为基本处理单位。

◇ Reader、Writer

　　◇ InputStreamReader、OutputStreamWriter

  ◇ PipedReader、PipedWriter

　　◇ FileReader、FileWriter

　　◇ CharArrayReader、CharArrayWriter

　　◇ FilterReader、FilterWriter

　　◇ BufferedReader、BufferedWriter

　　◇ StringReader、StringWriter

#### 字节流与字符流之间主要的区别

| 字节流                                                       | 字符流                                          |
| ------------------------------------------------------------ | ----------------------------------------------- |
| 基本单元为字节                                               | 基本单元为Unicode码元                           |
| 默认不使用缓冲区                                             | 使用缓冲区                                      |
| 通常用于处理二进制数据，实际上它可以处理任意类型的数据，但它不支持直接写入或读取Unicode码元 | 通常处理文本数据，它支持写入及读取Unicode码元。 |

**对象流**

　　◇ ObjectInputStream、ObjectOutputStream

​      串行化：对象通过写出描述自己状态的数值来记述自己的过程叫串行化

​      对象流：能够输入输出对象的流

​      将串行化的对象通过对象流写入文件或传送到其他地方，对象流是在普通流上加了传输对象的功能，所以构造对象流时要先构造普通文件流

​      注意：只有实现了Serializable接口的类才能被串行化）

#### BIO,NIO,AIO 有什么区别?

- BIO：Block IO 同步阻塞式 IO，就是我们平常使用的传统 IO，它的特点是模式简单使用方便，并发处理能力低。
- NIO：Non IO 同步非阻塞 IO，是传统 IO 的升级，客户端和服务器端通过 Channel（通道）通讯，实现了多路复用。
- AIO：Asynchronous IO 是 NIO 的升级，也叫 NIO2，实现了异步非堵塞 IO ，异步 IO 的操作基于事件和回调机制。

### 泛型

#### 泛型擦除

Java中的泛型，不是真正的泛型，只在源码中存在，在编译后的字节码文件中，就已经替换为原来的原生类型。

```java 
public static void main(String[] args){
	Map<String,String> map = new HashMap<String,String>();
	map.put("abc","123");
	map.put("efg","345");
	System.out.println(map.get("abc"));
	System.out.println(map.get("efg"));
}
```

编译后：

```java 
public static void main(String[] args){
	Map map = new HashMap();
	map.put("abc","123");
	map.put("efg","345");
	System.out.println((String)map.get("abc"));
	System.out.println((String)map.get("efg"));
}
```

当泛型遇到方法重载

例：

```java
public static GenericTypes {
  
    public static void method(List<String> list){

    }
  
    public static void method(List<Integer> list){

    }

}
```

上方两个方法， 在擦除后，变成了一样的远程类型List<E>，导致两种方法的特征签名一模一样，所以编译无法通过。

#### PECS

```java
	//PE
    //List<? super String>    String的父类√
    //List<? super CharSequence> CharSequence的父类  ×
    // CS
    //List<? extends CharSequence>  CharSequence的子类

//PE
  List<CharSequence> strList = new ArrayList<>();
  List<? super String> list = strList; // √ 编译通过。因为满足CharSequence是String的父类。

 List<String> strList1 = new ArrayList<>();
 List<? super CharSequence> list1 = strList1; // × 编译不过。因为因为T(String)不为CharSequence的父类。

//CS
 List<String> strList1 = new ArrayList<>();
 List<? extends CharSequence> list1 = strList1; // √ 编译通过。因为满足String是CharSequence的子类。

 List<CharSequence> strList1 = new ArrayList<>();
 List<? extends String> list1 = strList1;// × 编译不过。因为String不是CharSequence的子类。

```
#### kotlin泛型

inline 内联函数：代码平铺

reified 使用泛型类型关键字

kotlin泛型被称为**真泛型的原因**：

编译后，会获取到平铺代码对应的泛型真实类型，使用真实类型生成java代码



### 代理

举个栗子，

![img](https://tva1.sinaimg.cn/large/007S8ZIlly1ghsrydohxsj30wu05ujs4.jpg)

#### 静态代理

- 源码级：手动编写代理类、APT生成代理类

- 字节码级：编译期生成字节码、

  ```java
  interface 赚钱 {
      void makeMoney(int income);
  }
  
  class 小鲜肉 implements 赚钱 { //委托类
      @Override
      public void makeMoney(int income) {
          System.out.println("开拍，赚个" + income);
      }
  }
  
  class 经纪人 implements 赚钱 { //代理类
      赚钱 xxr;
  
      public 经纪人(赚钱 xxr) {
          this.xxr = xxr;
      }
  
      @Override
      public void makeMoney(int income) {
          if (income < 1000_0000) { //控制访问
              System.out.println("才" + income + "，先回去等通知吧");
          } else {
              xxr.makeMoney(income);
          }
      }
  }
  
  public static void main(String[] args) {
      赚钱 xxr = new 小鲜肉();
      赚钱 jjr = new 经纪人(xxr);
      jjr.makeMoney(100_0000); //输出：才1000000，先回去等通知吧
      jjr.makeMoney(1000_0000); //输出：开拍，赚个10000000
  }
  
  ```

  

#### 动态代理

运行期生成字节码，如Proxy.newProxyInstance。Proxy.newProxyInstance是java自带，只能对接口代理（因为生成的类已经继承了Proxy，java没法多继承）

```java
class 合作标准 implements InvocationHandler {
    赚钱 xxr;

    public 合作标准(赚钱 xxr) {
        this.xxr = xxr;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        int income = (int) args[0];
        if (income < 1000_0000) { //控制访问
            System.out.println("才" + income + "，先回去等通知吧");
            return null;
        } else {
            return method.invoke(xxr, args);
        }
    }
}

public static void main(String[] args) {
    赚钱 xxr = new 小鲜肉();
    合作标准 standard = new 合作标准(xxr);
    //生成类（字节码）：class $Proxy0 extends Proxy implements 赚钱
    //然后反射创建其实例bd，即来一场临时的商务拓展
    赚钱 bd = (赚钱) Proxy.newProxyInstance(赚钱.class.getClassLoader(),
                                        new Class[]{赚钱.class}, 
                                        standard);
    //调用makeMoney，内部转发给了合作标准的invoke
    bd.makeMoney(100_0000);
    bd.makeMoney(1000_0000);
}
```



### HashMap

HashMap 是基于 Map 接口实现的一种键-值对<key,value>的存储结构，允许 null 值，同时非有序，非同步(即线程不安全)。HashMap 的底层实现是数组 + 链表 + 红黑树（JDK1.8 增加了红黑树部分）。

HashMap 定位元素位置是通过键 key 经过扰动函数扰动后得到 hash 值， 然后再通过 hash & (length - 1)代替取模的方式进行元素定位的。

HashMap 是使用链地址法解决 hash 冲突的，当有冲突元素放进来时，会将此元素插入至此位置链表的最后一位，形成单链表。当存在位置的链表 长度大于等于 8 时，HashMap 会将链表 转变为 红黑树，以此提高查找效率。

HashMap 的容量是 2 的 n 次方，有利于提高计算元素存放位置时的效率， 

也降低了 hash 冲突的几率。

HashMap 不是线程安全的，Hashtable 则是线程安全的。但 Hashtable 是一个遗留容器，如果我们不需要线程同步，则建议使用 HashMap，如果需要线程同步，如果想线程安全，可以通过调用synchronizedMap(Map<K,V> m)使其线程安全。但是使用时的运行效率会下降，所以建议使用 ConcurrentHashMap 容器以此达到线程安全。

在多线程下操作 HashMap，由于存在扩容机制，当 HashMap 调用 resize() 进行自动扩容时，可能会导致死循环的发生。 

**红黑树部分问题：**

当HashMap中有大量的元素都存放到同一个桶中时，这个桶下有一条长长的链表，这个时候HashMap就相当于一个单链表，假如单链表有n个元素，遍历的时间复杂度就是O(n)，完全失去了它的优势。

针对这种情况，JDK1.8中引入了红黑树来优化这个问题，为什么不引入二叉查找树呢？因为二叉查找树的一般操作的执行时间为O(lgn)，但是二叉查找树若退化成了一棵具有n个结点的线性链后，则这些操作最坏情况运行时间为O(n)。与单链表一样。

所以此时我们需要红黑树它在二叉查找树的基础上增加了着色和相关的性质使得红黑树相对平衡，从而保证了红黑树的查找、插入、删除的时间复杂度最坏为O(log n)。

**为什么超过8才转换为红黑树**

根据概率统计决定的

理想情况下，在随机哈希码下，哈希表中节点的频率遵循泊松分布，而根据统计，忽略方差，列表长度为K的期望出现的次数是以上的结果，可以看到其实在为8的时候概率就已经很小了，再往后调整并没有很大意义。

####  HashMap 与 HashTable 对比

HashMap 是非 synchronized 的，性能更好，HashMap 可以接受为 null 的 key-value，而 Hashtable 是线程安全的，比 HashMap 要慢，不接受 null 的 key-value。

**HashMap key 为null时，其对应的hash值是0**



### LinkedHashMap （lrucache算法背后）

***HashMap+Double LinkedList***

- LinkedHashMap 是一个双向的链表，包含了指向链表头、尾的指针：head 和 tail，正是由于这个特性，所以LinkedHashMap 是有序的，而HashMap是无序的。
- LinkedHashMap 存储的Entey对象和HashMap的有不一样，因为是双向链表，所以肯定存在指向上个Node的指针 before以及指向下一个Node的指针 after
- LinkedHashMap 还有一个特有的变量 accessOrder，他是布尔型变量，true 表示链表按访问顺序，false 表示按插入顺序，

**lrucache算法背后使用的数据结构就是linkedhashmap**

### ConcurrentHashMap

ConcurrentHashMap(Base 1.7) 最外层不是一个大的数组，而是一个 Segment 的数组。每个 Segment 包含一个与 HashMap 数据结构差不多的链表数组。

- 是线程安全的Map，可针对每一个Segmen（继承自ReentrantLock） 单独上锁，比整体上锁效率更高。

- ConcurrentHashMap 的 get 方法是非常高效的，**因为整个过程都不需要加锁**。

1.8中，抛弃了原有的 Segment 分段锁，而采用了 CAS + synchronized 来保证并发安全性。底层实现是数组 + 链表 + 红黑树



### LinkedList

实现了 List 接口的双向链表。实现了所有可选列表操作，并且可以存储所有类型的元素，包括 null。 

LinkedList 是非同步的。若需要同步处理，应使Collections.synchronizedList 来包装

 

### CopyOnWriteArrayList

CopyOnWriteArrayList 是线程安全容器(相对于 ArrayList)，增加删除等写操作通过加锁的形式保证数据一致性，通过复制新集合的方式解决遍历迭代的问题。

上方有说明

### HashSet

（1）HashSet 内部使用 HashMap 的 key 存储元素，以此来保证元素不重复； 

（2）HashSet 是无序的，因为 HashMap 的 key 是无序的； 

（3）HashSet 中允许有一个 null 元素，因为 HashMap 允许 key 为 null； 

（4）HashSet 是非线程安全的； 

（5）HashSet 是没有 get()方法的；



### String、StringBuffer、StringBuilder

-  String 是 final 类，不能被继承。对于已经存在的 Stirng 对象，修改它的值，就是重新创建一个对象
- StringBuffer 是一个类似于 String 的字符串缓冲区，使用 append() 方法修改 Stringbuffer 的值，使用 toString() 方法转换为字符串，是线程安全的
- StringBuilder 用来替代于 StringBuffer，StringBuilder 是非线程安全的，速度更快


