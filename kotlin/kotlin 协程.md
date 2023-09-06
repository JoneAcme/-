# kotlin协程

[TOC]



## 一. 概念

### 1.  进程、线程与协程		

|          | 介绍                       | 说明                                         |
| -------- | -------------------------- | -------------------------------------------- |
| **进程** | 通常一个应用就是一个进程   | 一个进程中可以有多个线程，并且至少有一个线程 |
| **线程** | 操作系统调度的最小任务单位 | 抢占式多任务，调度完全由系统决定             |
| **协程** | 一个线程中可以有多个协程   | 调度完全由开发者控制                         |

> 注：由于 Kotlin 语言是基于 JVM 的，而 JVM 底层并没有对协程的支持，所以 Kotlin 中的协程本质上还是靠线程池实现的。但协程和线程并不是同一级别的概念，在其他语言中的协程可能与线程完全不一样，Kotlin 以线程池实现协程实属无奈之举。但在开发层面上，Kotlin 中的协程使用起来和其他语言的协程是类似的。

### 2. 协程优势

- 便捷：协程使得一个函数可以自动挂起和恢复，在进行耗时任务时，不必使用接口回调的方式等待耗时任务的结果。这种陈述式的编程，符合人类串行思维，避免了回调地狱。

- 高效：前文已提到过，协程比线程更轻量，性能更好。

- 灵活：由于协程的调度由开发者控制，使用起来更为灵活。

## 二.  作用域

| 作用域         | 说明                                                         |
| -------------- | ------------------------------------------------------------ |
| GlobalScope    | 顶级协程。生命周期是进程级别的，只要应用进程还在运行，即使 Activity 或 Fragment 被销毁，此作用域下的协程仍然会执行。 |
| MainScope      | 只能在 Activity 中使用，使用 MainScope 时，需要在 onDestroy 中调用 cancel() 取消协程。 |
| viewModelScope | 只能在 ViewModel 中使用，它会绑定 ViewModel 的生命周期       |
| lifecycleScope | 只能在 Activity、Fragment 中使用，它会绑定 Activity 和 Fragment 的生命周期。 |

GlobalScope：

```kotlin
fun main() {
    GlobalScope.launch {
        println("coroutine scope")
    }
    // 等待一秒钟保证协程运行完毕。
    Thread.sleep(1000)
}
```

Android MainScope：

```kotlin
class MainActivity : AppCompatActivity() {
    private val mainScope = MainScope()
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        mainScope.launch {
            // do something
        }
    }

    override fun onDestroy() {
        super.onDestroy()
        mainScope.cancel()
    }
}
```

也可以使用 Kotlin 中的 by 关键字，通过代理模式将 MainActivity 变成 MainScope：

```kotlin
class MainActivity : AppCompatActivity(), CoroutineScope by MainScope() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        launch {
            // do something
        }
    }

    override fun onDestroy() {
        super.onDestroy()
        cancel()
    }
}
```

## 三. 启动协程

| 启动方式    | 描述                                                    |
| ----------- | ------------------------------------------------------- |
| launch      | 启动的协程不带有返回值。                                |
| async       | 以最后一行作为返回值，通过 await 可以获得其返回值。     |
| withContext | 以最后一行作为返回值，可指定调度器，与async串行执行一致 |

------

1. **launch**

```kotlin
fun main() = runBlocking<Unit> {
    val start = System.currentTimeMillis()
    var valueA = 0;
    var valueB = 0;
    val a = launch {
        delay(1000)
        valueA++
    }
    val b = launch {
        delay(2000)
        valueB += 2
    }
    println("1. The answer is ${valueA * valueB}")
    a.join()
    println("2. The answer is ${valueA * valueB}")
    b.join()
    println("3. The answer is ${valueA * valueB}")
    println(System.currentTimeMillis() - start)
}	
```

输出：

```
1. The answer is 0
2. The answer is 0
3. The answer is 2
2024
```

------

2.  **async**

```kotlin
fun main() = runBlocking<Unit> {    
		val start = System.currentTimeMillis()
    val a = async {
        delay(1000)
        println("I'm computing part of the answer")
        6
    }
    val b = async {
        delay(1000)
        println("I'm computing another part of the answer")
        7
    }
    println("The answer is ${a.await() * b.await()}")
    println(System.currentTimeMillis() - start)
}
```

输出：

```
I'm computing part of the answer
I'm computing another part of the answer
The answer is 42
1020
```

**如果 async 开启协程后，马上调用 await()， 会让协程串行执行**：

```kotlin
fun main() = runBlocking<Unit> {
    val start = System.currentTimeMillis()
    val a = async {
        delay(1000)
        println("I'm computing part of the answer")
        6
    }.await()
    val b = async {
        delay(1000)
        println("I'm computing another part of the answer")
        7
    }.await()
    println("The answer is ${a * b}")
    println(System.currentTimeMillis() - start)
}
```

输出：

```
I'm computing part of the answer
I'm computing another part of the answer
The answer is 42
2030
```

------

3. **withContext**

```kotlin
fun main() = runBlocking<Unit> {
    val start = System.currentTimeMillis()
    val a = withContext(Dispatchers.Default) {
        delay(1000)
        6
    }
    val b = withContext(Dispatchers.Default) {
        delay(2000)
        7
    }
    println(a*b)
    println(System.currentTimeMillis() - start)
}
```

输出

```
42
3052
```

## 四. 调度器

| 调度器              | 说明                                                         |
| ------------------- | ------------------------------------------------------------ |
| Dispatchers.Default | 非主线程，用于执行 CPU 密集型任务。常见的 CPU 密集型任务有数组排序、Json 数据解析等。 |
| Dispatchers.IO      | 非主线程，用于执行 IO 密集型任务。常见的 IO 密集型任务有数据库读写、文件读写、网络请求等。 |
| Dispatchers.Main    | 专用于 Android 主线程，用于处理 UI 交互或一些轻量级任务，如展示 UI 控件，更新 LiveData 等。 |

## 五. 启动模式

| 模式         | 说明                                                         |
| ------------ | ------------------------------------------------------------ |
| DEFAULT      | 立即开始调度，协程在调度前可以被取消。                       |
| ATOMIC       | 立即开始调度，协程执行到第一个挂起点之前不响应取消。         |
| LAZY         | 只有协程被需要时，包括主动调用协程的 start()、join()、await() 等函数时才会开始调度，协程在调度前可以被取消。 |
| UNDISPATCHED | 立即在当前函数调用栈中**执行**，直到遇到第一个挂起点后才会切换到协程上下文中执行。 |

**调度： 处于等待执行状态，即开始抢占CPU，等待执行。**

------

1. **DEFAULT**

```kotlin
fun main() = runBlocking<Unit> {
    val job = launch(start = CoroutineStart.DEFAULT) {
        println("start")
        delay(5000)
        println("done")
    }
    job.cancel()
}
```

以上代码没有任何输出，执行前就被取消了。

------

2. **ATOMIC**

```kotlin
fun main() = runBlocking<Unit> {
    val job = launch(start = CoroutineStart.ATOMIC) {
        println("start")
        delay(5000)
        println("done")
    }
    job.cancel()
}
```

输出：

```
start
```

协程开始执行前，不响应`cancel()`。`delay()` 函数是一个挂起函数，所以这里就是一个挂起点，协程运行到这里的时候才响应了取消命令。

------

3. **LAZY**

```kotlin
fun main() = runBlocking<Unit> {
    val job = launch(start = CoroutineStart.LAZY) {
        println("start")
        delay(5000)
        println("done")
    }
    job.start()
}
```

如果不调用 job.start()，将不会有任何输出。只有调用了 job.start() 后，程序才能正常执行。

------

4. **UNDISPATCHED**

```kotlin
fun main() = runBlocking<Unit> {
    launch(context = Dispatchers.IO, start = CoroutineStart.UNDISPATCHED) {
        println("start ${Thread.currentThread().name}")
        delay(5000)
        println("done ${Thread.currentThread().name}")
    }
}
```

输出：

```
start main
done DefaultDispatcher-worker-1
```

*立即在当前函数调用栈中**执行**，直到遇到第一个挂起点后才会切换到协程上下文中执行。* **这里注意是立即执行，不是调度。**

## 六. Job

每一个协程创建时，都会生成一个 Job 实例，这个实例是协程的唯一标识，负责管理协程的生命周期。
当协程创建、执行、完成、取消时，Job 的状态也会随之改变，通过检查 Job 对象的属性可以追踪到协程当前的状态。

### 所有状态

- 新创建 New
- 活跃 Active
- 完成中 Completing
- 已完成 Completed
- 取消中 Cancelling
- 已取消 Cancelled

![](https://i.imgur.com/VC01jcY.png)

### 取消

- 对协程作用域调用 cancel() 函数会取消此作用域内的所有子协程
- 取消单个子任务，可以调用单个子任务的 cancel() 函数，取消单个子协程不会影响其兄弟协程继续执行

取消时，会外抛`CancellationException`异常，但是**可不捕获，不影响程序执行**（协程的作用域认为 CancellationException 异常是一个「正常」的异常，所以并没有往外继续抛出，作用域直接将其静默处理掉了。）。

```kotlin
fun main() = runBlocking<Unit> {
    val job = launch(start = CoroutineStart.ATOMIC) {
        try {
            println("start")
            delay(500)
            println("done")
        } catch (e: CancellationException) {
            e.printStackTrace()
        }
    }
    job.cancel()
}	
```

使用启动模式为`CoroutineStart.ATOMIC`，保证协程执行，输出：

```
start
kotlinx.coroutines.JobCancellationException: StandaloneCoroutine was cancelled; job=StandaloneCoroutine{Cancelling}@6f79caec
```

**如果要协程不响应`cancel()`, 可以使用 `WithContext(NonCancellable)`**

### 状态判断

**活跃状态**： 协程取消后， Job 是否处于活跃状态（Job 的 isActive 属性为 false）。

| 活跃状态判断   | 说明                                                         |
| -------------- | ------------------------------------------------------------ |
| isActive       | 参考上方状态图，New、Completed、Cancelled 时，该属性为false  |
| ensureActive() | 非活跃状态，会立即抛出 CancellationException 异常。          |
| yield()        | 非活跃状态，会立即抛出 CancellationException 异常，同时会尝试让出线程的执行权。 |

Job 取消的案例：
1. Job是一个 CPU 密集型任务，不断循环。
2. 每隔500ms打印一次，打印5次后退出。
3. 在1200ms时，调用`cancelAndJoin()`取消协程

```kotlin
fun main() = runBlocking<Unit> {
    val startTime = System.currentTimeMillis()
    val job = launch(Dispatchers.Default) {
        var nextPrintTime = startTime
        var i = 0
        while (i < 5) {
            if (System.currentTimeMillis() >= nextPrintTime) {
                println("I'm sleeping $i")
                i++
                nextPrintTime += 500
            }
        }
    }
    delay(1200)
    println("Time to finish") 
    job.cancelAndJoin()
    println("Done")
}
```

预计输出： 执行三次打印，cancel执行

```
I'm sleeping 0
I'm sleeping 1
I'm sleeping 2
Time to finish
```

实际输出：**执行完所有的五次打印才退出。**

```
I'm sleeping 0
I'm sleeping 1
I'm sleeping 2
Time to finish
I'm sleeping 3
I'm sleeping 4
Done
```

**原因： Job 是一个 CPU 密集型任务，一直抢占着 CPU，取消时不会释放资源。**

**解决：**在循环时候，判断一下当前协程的活跃状态。

```kotlin
fun main() = runBlocking<Unit> {
    val startTime = System.currentTimeMillis()
    val job = launch(Dispatchers.Default) {
        var nextPrintTime = startTime
        var i = 0
        while (i < 5 && isActive) {
            if (System.currentTimeMillis() >= nextPrintTime) {
                println("I'm sleeping $i")
                i++
                nextPrintTime += 500
            }
        }
    }
    delay(1200)
    println("Time to finish")
    job.cancelAndJoin()
    println("Done")
}
```

输出：

```
I'm sleeping 0
I'm sleeping 1
I'm sleeping 2
Time to finish
Done
```

这里也可以使用 `ensureActive()` 或 `yield()`，也可以达到同样效果：

```kotlin
while (i < 5) {
    ensureActive()
    ...
}
```

或者

```kotlin
while (i < 5) {
    yield()
    ...
}
```

*`ensureActive()` 或 `yield()` 会检查 Job 任务当前的状态，如果是非活跃状态，协程就会抛出 CancellationException 异常，不再继续执行了。*

### 资源释放

Job 在cancel() 时协程中的代码块会收到 CancellationException，所以可以在`finally`中释放资源。

```kotlin
fun main() = runBlocking<Unit> {
    val job = launch {
        try {
            delay(1000)
        } catch (e: CancellationException) {
            e.printStackTrace()
        }finally {
            //  todo 这里释放资源
            println("release resource")
        }
    }
    yield() // 保证job执行 与 delay(500) 或者 指定启动模式为 CoroutineStart.ATOMIC 效果类似 
    job.cancelAndJoin()
}
```

这里，如果释放资源比较耗时：

```kotlin
fun main() = runBlocking<Unit> {
    val job = launch {
        try {
            delay(1000)
        } catch (e: CancellationException) {
            e.printStackTrace()
        } finally {
            //  todo 这里释放资源
            println("finally start")
            delay(1000)
            println("finally end")
        }
    }
    yield()
    job.cancelAndJoin()
}
```

输出：

```
finally start
kotlinx.coroutines.JobCancellationException: StandaloneCoroutine was cancelled; job=StandaloneCoroutine{Cancelling}@5d3411d
```

这里`finally` 中的 `delay()` 函数并没有成功挂起，协程直接被 cancel 掉了。所以这种方案不可行。

想要挂起成功，需要使用`WithContext(NonCancellable)`

```kotlin
fun main() = runBlocking<Unit> {
    val job = launch {
        try {
            delay(1000)
        } catch (e: CancellationException) {
            e.printStackTrace()
        }finally {
            println("finally start")
            withContext(NonCancellable) {
                println("delay start")
                delay(1000)
                println("delay end")
            }
            println("finally end")
        }
    }
    yield()
    job.cancelAndJoin()
}
```

输出：

```
kotlinx.coroutines.JobCancellationException: StandaloneCoroutine was cancelled; job=StandaloneCoroutine{Cancelling}@1ef7fe8e
finally start
delay start
delay end
finally end
```

### 超时处理

| 指定超时          | 描述                                                         |
| ----------------- | ------------------------------------------------------------ |
| withTimeout       | 函数会在超时后抛出一个超时异常 TimeoutCancellationException  |
| withTimeoutOrNull | 正常：会将代码块最后一行作为返回值； 超时：返回一个 null 值； |

**withTimeout**：

```kotlin
fun main() = runBlocking<Unit> {
    try {
        withTimeout(500) {
            launch {
                delay(1000)
            }
        }
    } catch (e: TimeoutCancellationException) {
        e.printStackTrace()
    }
}
```

```
kotlinx.coroutines.TimeoutCancellationException: Timed out waiting for 500 ms
```

**withTimeoutOrNull**：

超时：

```kotlin
fun main() = runBlocking<Unit> {
    val result = withTimeoutOrNull(500) {
				delay(1000)
    }
    println("result: $result")
}
```

```
result: null
```

正常：

```kotlin
fun main() = runBlocking<Unit> {
    val result = withTimeoutOrNull(2000) {
        delay(1000)
        123
    }
    println("result: $result")
}
```

```
123
```

## 七. CoroutineContext--上下文

### 组成

| 属性                      | 说明                                 |
| ------------------------- | ------------------------------------ |
| Job                       | 控制协程生命周期                     |
| CoroutineDispatcher       | 分发线程，默认 Dispatchers.Default。 |
| CoroutineName             | 协程名称，默认值是 "coroutine"。     |
| CoroutineExceptionHandler | 指定未被捕获的异常的处理器。         |

指定名称（重载了`plus`，这里直接使用 + ）：

```kotlin
fun main() = runBlocking<Unit> {
    launch(Dispatchers.Default + CoroutineName("hello")) {
        println("Current thread: ${Thread.currentThread().name}")
        println("this.name:${this.coroutineContext[CoroutineName]}")
    }
}
```

```
Current thread: DefaultDispatcher-worker-1
this.name:CoroutineName(hello)
```

**新创建的协程，它的 CoroutineContext 会包含一个全新的 Job 实例，其他元素从CoroutineContext父类继承，而父类也可能是另一个协程或者CoroutineScope。每个协程的上下文中，Job 对象一定是不一样的，而其他元素是继承过来的，都是同一个对象。**

### 异常处理

| 异常分类                                                     | 说明         | 当前为根协程                                                 | 当前非根协程 |
| ------------------------------------------------------------ | ------------ | ------------------------------------------------------------ | ------------ |
| 自动传播异常（[launch](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/launch.html) 与 [actor](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/actor.html)） | 协程内部异常 | 视为**未捕获**异常，类似 Java 的 `Thread.uncaughtExceptionHandler` |              |
| 向用户暴露异常（[async](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/async.html) 与 [produce](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/produce.html)） | 用户调用抛出 | 依赖用户最终消费异常                                         |              |



异常示例：

```kotlin
fun main() = runBlocking<Unit> {
	val job = GlobalScope.launch {
        try {
            throw Exception()
        } catch (e: Exception) {
            println("catch in coroutine")
        }
    }
    job.join()

    // 2. 用户消费异常:
    val deferred = GlobalScope.async {
        throw Exception()
    }
    try {
        deferred.await()
    } catch (e: Exception) {
        println("catch in await")
    }
}
```

