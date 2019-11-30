[TOC]

------

## 简介

**提升CPU利用率->减少线程切换->提高运行效率**

线程发生阻塞时，主动交出CPU，阻塞完成后，继续获取CPU。

线程的锁获取、释放都会耗费很大的资源。协程可以暂时挂起，消耗小得多。

协程本身不限制运行在线程中，可直接运行在进程中，目前的编程语言环境，导致都是多个线程运行，也就是协程运行在线程中，但协程和线程本质上并没有依赖关系。

一个线程中可有多个协程。

**特点**：

1. 可控制：可控制发起子任务，包括启动、停止
2. 轻量级：占用资源比线程少
3. 语法糖：多任务多线程避免各种回调，导致密之缩进

## 启动协程

**启动方式：**

1. `runBloking : T` ：通常用于启动最外层协程，线程切换到协程等，会导致阻塞。
2. `launch : Job`：用于执行协程任务
3. `async/await : Deferred`：用于异步执行任务，并得到返回值

```kotlin
fun main(args: Array<String>) = runBlocking {

    val job = launch {
        println("launching")
        delay(2500)
    }

    val async = async {
        println("async")
        delay(2000)
        return@async "async return"
    }
    println(async.await())
    println("ending")
}
```



## 协程的参数

以`launch`为例：

```kotlin
public fun CoroutineScope.launch(
    context: CoroutineContext = EmptyCoroutineContext,
    start: CoroutineStart = CoroutineStart.DEFAULT,
    block: suspend CoroutineScope.() -> Unit
): Job {
    val newContext = newCoroutineContext(context)
    val coroutine = if (start.isLazy)
        LazyStandaloneCoroutine(newContext, block) else
        StandaloneCoroutine(newContext, active = true)
    coroutine.start(start, coroutine, block)
    return coroutine
}
```

1.  `context: CoroutineContext = EmptyCoroutineContext`：协程上下文
   - 协程调度器以及传递参数作用
   - 指定协程执行、恢复所在线程
2. `start: CoroutineStart = CoroutineStart.DEFAULT`：启动方式
   - `DEFAULT`：有空时候就启动
   - `LAZY`：调用start或者`join`方法后才会启动
   - `ATOMIC`：以原子性质启动，一旦启动，不能cancel
   - `UNDISPATCHED`：与`DEFAULT`相同，也可用于自定义启动方式时传入
3.  `block: suspend CoroutineScope.() -> Unit`：要执行的闭包



## Android 中使用(待补充)

```kotlin
fun getXXX( tv:TextView) = runBlocking{
		launch(UI){
			tv.text = async{ 网络请求等...}.await();
		}
}
```



## suspend 协程函数

一个协程方法(或闭包)必须被 `suspend` 修饰，同时 `suspend` 修饰的方法(或闭包)只能调用被`suspend`修饰过的方法(或闭包)。因为`suspend` 修饰的方法编译后会多出一个`Continuation`类型参数。协程的本质，就是一次回调，是通过`Continuation`的接口回调实现的

`Continuation.kt`:

```kotlin
/**
 * Interface representing a continuation after a suspension point that returns a value of type `T`.
 */
@SinceKotlin("1.3")
public interface Continuation<in T> {
    
  // 执行的线程调度！~~~
    public val context: CoroutineContext

  // 执行的结果！~~
    public fun resumeWith(result: Result<T>)
}

@SinceKotlin("1.3")
@InlineOnly
public inline fun <T> Continuation<T>.resume(value: T): Unit =
    resumeWith(Result.success(value))

@SinceKotlin("1.3")
@InlineOnly
public inline fun <T> Continuation<T>.resumeWithException(exception: Throwable): Unit =
    resumeWith(Result.failure(exception))

```



`Result.kt`:

```kotlin
@SinceKotlin("1.3")
public inline class Result<out T> @PublishedApi internal constructor(
    @PublishedApi
    internal val value: Any?
) : Serializable {
    // discovery

    /**
     * Returns `true` if this instance represents successful outcome.
     * In this case [isFailure] returns `false`.
     */
    public val isSuccess: Boolean get() = value !is Failure

    /**
     * Returns `true` if this instance represents failed outcome.
     * In this case [isSuccess] returns `false`.
     */
    public val isFailure: Boolean get() = value is Failure
...
}
```



## Java 中使用协程

`suspend` 方法编译后会多出一个`Continuation` 类型参数，用于回调使用，所以在java调用`suspend`方法时，需要显式传入一个`Continuation` 接口的实现对象，用于协程执行结果的调用。

APITest.kt

```kotlin
suspend  fun getResult(tv: TextView) = runBlocking {
    tv.text = withContext(Dispatchers.Main) {
        //模拟网络请求。。。。
        delay(500)
       "this is result"
    }
}
```

Text.java

```java
public class Test {
    public void getResult(TextView textView) {
        APITestKt.getResult(textView, new Continuation<Unit>() {
            @NotNull
            @Override
            public CoroutineContext getContext() {
              return   Dispatchers.getMain();
            }

            @Override
            public void resumeWith(@NotNull Object o) {
                if (o instanceof Result) {
                    Result result = (Result) o;
                    //...
                }
            }
        });
    }
}
```



## 特殊的启动协程的方式

1. buildSequence/yield : Sequence

   用于执行会多次返回数据的协程任务

2. buildIterator : Iterator

   与buildSequence/yield类似，只是返回值不同，后者也可拿到迭代器对象，通常不适用

3. produce : Channel 

   用于执行协程任务，并得到一个channel

## Channel 协程间通信

​	生产者消费者模式。可声明缓存池存放数量，写入时已满、取出时为空，则挂起。

​	cancel后不可继续使用，数据不保存。

```kotlin
fun main() = runBlocking {

    fun sended() = produce {
        for (i in 0..10)
            send(i)
    }

    val channel = sended()
    channel.consumeEach {
        println(it)
    }

}
```



## BIO/NIO

1. BIO：Blocking IO 同步阻塞IO

2. NIO：Non-Blocking IO 同步不阻塞IO，基于操作缓存区实现

Java 写法

 ```java
public static void copyFileBIO(File source, File desk) {
        FileInputStream inputStream = null;
        FileOutputStream outputStream = null;
        try {
            inputStream = new FileInputStream(source);
            outputStream = new FileOutputStream(desk);

            byte[] buff = new byte[1024];
            int lenth;

            while ((lenth = inputStream.read(buff)) > 0) {
                outputStream.write(buff, 0, lenth);
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            close(inputStream, outputStream);
        }

    }

    public static void copyFileNio(File source, File desk) {
        FileChannel inputCannel = null;
        FileChannel outChannel = null;
        try {
            inputCannel = new FileInputStream(source).getChannel();
            outChannel = new FileOutputStream(desk).getChannel();

            /***
             * 写入的两种方式，两者等同
             */
            // 1. 自己实现缓存区
            ByteBuffer allocate = ByteBuffer.allocate(1024);
            while (true) {
                allocate.clear();

                if (inputCannel.read(allocate) <= 0) {
                    break;
                }
                outChannel.write(allocate);
            }
            // 2. 调用sdk中的实现
            outChannel.transferFrom(inputCannel, 0, inputCannel.size());

        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            close(inputCannel, outChannel);
        }

    }


    public static void close(Closeable... closes) {
        for (Closeable closeable : closes) {
            try {
                if (closeable == null) continue;
                closeable.close();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }

 ```

kotlin写法

