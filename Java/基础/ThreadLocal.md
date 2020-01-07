# ThreadLocal

[TOC]

## 一.介绍：

```java

/**
 * This class provides thread-local variables.  These variables differ from
 * their normal counterparts in that each thread that accesses one (via its
 * {@code get} or {@code set} method) has its own, independently initialized
 * copy of the variable.  {@code ThreadLocal} instances are typically private
 * static fields in classes that wish to associate state with a thread (e.g.,
 * a user ID or Transaction ID).
 * 
 * <p>For example, the class below generates unique identifiers local to each
 * thread.
 * A thread's id is assigned the first time it invokes {@code ThreadId.get()}
 * and remains unchanged on subsequent calls.
 */     
    
```

ThreadLocal提供了线程的局部变量，每个线程都可以通过`set()`和`get()`来对这个局部变量进行操作，但不会和其他线程的局部变量进行冲突，**实现了线程的数据隔离**～。

简要言之：往ThreadLocal中填充的变量属于**当前**线程，该变量对其他线程而言是隔离的。

## 二.使用举例

### 1.管理数据库的Connection：

   当时在学JDBC的时候，为了方便操作写了一个简单数据库连接池，需要数据库连接池的理由也很简单，频繁创建和关闭Connection是一件非常耗费资源的操作，因此需要创建数据库连接池～

那么，数据库连接池的连接怎么管理呢？？我们交由ThreadLocal来进行管理。为什么交给它来管理呢？？ThreadLocal能够实现**当前线程的操作都是用同一个Connection，保证了事务！

### 2.Android looper

Looper类就是利用了ThreadLocal的特性，保证每个线程只存在一个Looper对象。

```java
static final ThreadLocal<Looper> sThreadLocal = new ThreadLocal<Looper>();
private static void prepare(boolean quitAllowed) {
    if (sThreadLocal.get() != null) {
        throw new RuntimeException("Only one Looper may be created per thread");
    }
    sThreadLocal.set(new Looper(quitAllowed));
}
```



### 3.Android EventBus中：

   通过ThreadLocal存储各个线程中的变量。

```java
private final ThreadLocal<PostingThreadState> currentPostingThreadState = new ThreadLocal<PostingThreadState>() {
    @Override
    protected PostingThreadState initialValue() {
        return new PostingThreadState();
    }
};
```

```java
final static class PostingThreadState {
    //通过post方法参数传入的事件集合
    final List<Object> eventQueue = new ArrayList<Object>(); 
    boolean isPosting; //是否正在执行postSingleEvent()方法
    boolean isMainThread;
    Subscription subscription;
    Object event;
    boolean canceled;
    }
```

## 三.ThreadLocal实现的原理

### 1.源码阅读

```java

    public void set(T value) {

        // 得到当前线程对象
        Thread t = Thread.currentThread();
        
        // 这里获取ThreadLocalMap
        ThreadLocalMap map = getMap(t);

        // 如果map存在，则将当前线程对象t作为key，要存储的对象作为value存到map里面去
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
    }

```

ThreadLocalMap：

```java

static class ThreadLocalMap {

        /**
         * The entries in this hash map extend WeakReference, using
         * its main ref field as the key (which is always a
         * ThreadLocal object).  Note that null keys (i.e. entry.get()
         * == null) mean that the key is no longer referenced, so the
         * entry can be expunged from table.  Such entries are referred to
         * as "stale entries" in the code that follows.
         */
        static class Entry extends WeakReference<ThreadLocal<?>> {
            /** The value associated with this ThreadLocal. */
            Object value;

            Entry(ThreadLocal<?> k, Object v) {
                super(k);
                value = v;
            }
        }
        //....很长
}
```

**ThreadLocalMap是ThreadLocal的一个内部类。用Entry类来进行存储**，我们的**值都是存储到这个Map上的，key是当前ThreadLocal对象**。

如果该Map不存在，则初始化一个：

```java

    void createMap(Thread t, T firstValue) {
        t.threadLocals = new ThreadLocalMap(this, firstValue);
    }
```

如果该Map存在，则**从Thread中获取**

```java

    /**
     * Get the map associated with a ThreadLocal. Overridden in
     * InheritableThreadLocal.
     *
     * @param  t the current thread
     * @return the map
     */
    ThreadLocalMap getMap(Thread t) {
        return t.threadLocals;
    }

```

Thread维护了ThreadLocalMap变量

```java

    /* ThreadLocal values pertaining to this thread. This map is maintained
     * by the ThreadLocal class. */
    ThreadLocal.ThreadLocalMap threadLocals = null
```

**ThreadLocalMap是在ThreadLocal中使用内部类来编写的，但对象的引用是在Thread中**！

总结：**Thread为每个线程维护了ThreadLocalMap这么一个Map，而ThreadLocalMap的key是LocalThread对象本身，value则是要存储的对象**

`get()`:

```java

    public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        return setInitialValue();
    }
```

### 2.原理总结

1. 每个Thread维护着一个ThreadLocalMap的引用

2. ThreadLocalMap是ThreadLocal的内部类，用Entry来进行存储

3. 调用ThreadLocal的set()方法时，实际上就是往ThreadLocalMap设置值，key是ThreadLocal对象，值是传递进来的对象

4. 调用ThreadLocal的get()方法时，实际上就是往ThreadLocalMap获取值，key是ThreadLocal对象

5. **ThreadLocal本身并不存储值**，它只是**作为一个key来让线程从ThreadLocalMap获取value**。

## 四.避免内存泄漏

对象关系引用图：

![](https://ws4.sinaimg.cn/large/006tNbRwly1fw93dfbl9nj30k00ckab7.jpg)

ThreadLocal内存泄漏的根源是：**由于ThreadLocalMap的生命周期跟Thread一样长，如果没有手动删除对应key就会导致内存泄漏，而不是因为弱引用**。

想要避免内存泄露就要**手动remove()掉**！

## 五.总结

**ThreadLocal设计的目的就是为了能够在当前线程中有属于自己的变量，并不是为了解决并发或者共享变量的问题**