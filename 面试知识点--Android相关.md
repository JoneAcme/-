### *面试知识点--Android相关**



[TOC]



## Activity 

[![img](https://camo.githubusercontent.com/c20011a1fbd0b014e39f49fc5b0a03c631433ce4/687474703a2f2f6769747975616e2e636f6d2f696d616765732f6c6966656379636c652f61637469766974792e706e67)](https://camo.githubusercontent.com/c20011a1fbd0b014e39f49fc5b0a03c631433ce4/687474703a2f2f6769747975616e2e636f6d2f696d616765732f6c6966656379636c652f61637469766974792e706e67)

**Activity A 启动另一个Activity B**，回调如下:

- Activity A#onPause() 
- Activity B#onCreate() 
- Activity B#onStart() 
- Activity B#onResume() 
- Activity A#onStop()；如果B是透明主题又或则是个DialogActivity，则不会回调A的onStop；

### 启动模式

| LaunchMode     | 说明                                                         |
| -------------- | ------------------------------------------------------------ |
| standard       | 标准模式，系统在启动它的任务中创建 activity 的新实例         |
| singleTop      | 栈顶复用模式，如果activity的实例已存在于当前任务的顶部，则系统通过调用其onNewIntent()，否则会创建新实例 |
| singleTask     | 栈内复用模式，如果栈中已有实例，则调用其 onNewIntent() 方法，其上面的实例会被移除栈（ClearTop）。如果不存在，这则新建一个想要的栈，并新建一个实例 |
| singleInstance | 相同 singleTask，activity始终是其task的唯一成员; 任何由此开始的activity 都在一个单独的 task 中打开 |

栈：

- 默认情况下，所有的Activity的任务栈都为应用的包名

- 可以为Activity单独指定TaskAffinity属性，指定栈。(TaskAffinity一般与singleTask或allowTaskReparenting属性配对使用，其他情况下没有意义)

  > allowTaskReparenting： 应用A 启动另一个应用B 的Activity C，如果allowTaskReparenting为True，那么此时点击应用B时，启动的是不是主Activity，而是之前启动的Activity C

### 保存状态

`onSaveInstanceState`/`onRestoreInstanceState`

只有Activity 是在异常状态下被销毁的，系统会调用`onSaveInstanceState`保存当前状态，它既**可能在`onPause`之前调用也可能在`onPause`之后调用**。当Activity被重建后，系统会调用`onRestoreInstanceState`，`onRestoreInstanceState`的调用时机在onStart之后。

onSaveInstanceState()会在以下情况被调用：
1、当用户按下HOME键时。
2、从最近应用中选择运行其他的程序时。
3、按下电源按键（关闭屏幕显示）时。
4、从当前activity启动一个新的activity时。
5、屏幕方向切换时(无论竖屏切横屏还是横屏切竖屏都会调用)。

## Fragment

[![img](https://camo.githubusercontent.com/d80aae526660b859bb231a9fc0bcc50fe55f8c7e/68747470733a2f2f646576656c6f7065722e616e64726f69642e676f6f676c652e636e2f696d616765732f667261676d656e745f6c6966656379636c652e706e67)



### setArguments

当一个fragment重新创建的时候，系统会再次调用 Fragment中的**默认构造函数**。 

如果创建了一个带有重要参数的Fragment的之后，一旦由于什么原因（横竖屏切换）导致Fragment重新创建。

此时，重建默认调用无参的构造函数，也就是通过构造方法传递的参数会丢失。

setArguments方法中传递的参数，可以保存，不会丢失

fragment 的状态会保存在FragmentState（implements Parcelable）中，在重建时候由FragmentManager从FragmentState取出数据重新设置回去

### FragmentTransaction管理的Fragment生命周期状态

#### add() VS replace()

只有在Fragment数量大于等于2的时候，调用add()还是replace()的区别才能体现出来。

- 通过add()连续两次添加Fragment的时候
  1. 每个Fragment生命周期中的onAttach()-onResume()都会被各调用一次，而且两个Fragment的View会被同时attach到containerView中
  2. 退出Activty时，每个Fragment生命周期中的onPause() - onDetach()也会被各调用一次。

- 使用replace()来添加Fragment的时候

  第二次添加的Fragment会导致第一个Fragment被销毁（在不使用回退栈的情况下），即执行第二个Fragment的onAttach()方法之前会先执行第一个Fragment的onPause()-onDetach()方法，与此同时containerView会detach掉第一个Fragment的View。

#### show() & hide() VS attach() & detach()

- 调用show() & hide()

  **Fragment的正常生命周期方法并不会被执行**，仅仅是Fragment的View被显示或者隐藏，并视情况调用onHiddenChanged()。而且，尽管Fragment的View被隐藏，但它在父布局中并未被detach，仍然是作为containerView的childView存在着。

- 调用attach() & detach() 

  一旦一个Fragment被detach()，它的onPause()-onDestroyView()周期都会被执行，同时Fragment的View也会被detach，但是**不会执行onDestroy()和onDetach()**，也就是说Fragment的实例还是在内存中的。

- remove()

  相对应add()方法执行onAttach()-onResume()的生命周期，remove()就是完成剩下的onPause()-onDetach()周期。而且replace()其实就是remove()+add()。

#### FragmentPagerAdapter

实际上是使用add()，attach()和detach()来管理Fragment的。缓冲范围内的从onAttach() - onResume()，超出缓存范围onPause() - onDestroyView()。

#### FragmentStatePagerAdapter

使用add()和remove()管理Fragment，所以缓存外的Fragment的实例不会保存在内存中





## Service

[![img](https://camo.githubusercontent.com/00a0bb98ef0d7d0f0539d5fc4d99e7c530401cf1/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f3934343336352d636635633161396432646464616163612e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f3435362f666f726d61742f77656270)](https://camo.githubusercontent.com/00a0bb98ef0d7d0f0539d5fc4d99e7c530401cf1/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f3934343336352d636635633161396432646464616163612e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f3435362f666f726d61742f77656270)

启动方式

- 直接开启startService，使用stopService关闭。
  stratService和stopService一一对应，一个开启，一个结束。
  1. startService
  2. onCreate
  3. onStartCommand
  4. StopService
  5. onDestroy
- 绑定开启bindService，使用unbindService解绑关闭。
  bindServic和unbindService一一对应，一个绑定开启，一个解绑结束。
  1. bindService
  2. onCreate
  3. onStartCommand
  4. onUnbind
  5. onDestroy

注意以下条件：
 1.每个Service只会存在一个实例。在整个生命周期内，只有**startCommand()**能被多次调用。其他方法只能被调用**一次**。（即只能绑定和解绑一次。）
 2.绑定后没有解绑，**无法**使用stopService()将其停止。
 3.如果已经onCreate()，那么startService()将**只**调用startCommand()。
 4.如果是以bindService开启，那么使用unbindService时就会**自动调用**onDestroy销毁。

## BroadcastReceiver

- Android8.0后，所有隐式广播都不允许使用静态注册的方式来接收了。

```java
LocalBroadcastManager.getInstance(MainActivity.this).registerReceiver(receiver, filter);
```

## ContentProvider

ContentProvider 管理对结构化数据集的访问。它们封装数据，并提供用于定义数据安全性的机制。 内容提供程序是连接一个进程中的数据与另一个进程中运行的代码的标准界面。

ContentProvider 的 onCreate 要先于 Application 的 onCreate 而执行。

## View

ViewRoot 对应于 ViewRootImpl 类，它是连接 WindowManager 和 DecorView 的纽带，View 的三大流程均是通过 ViewRoot 来完成的。在 ActivityThread 中，当 Activity 对象被创建完毕后，会将 DecorView 添加到 Window 中，同时会创建 ViewRootImpl 对象，并将 ViewRootImpl 对象和 DecorView 建立关联

View 的整个绘制流程可以分为以下三个阶段：

- measure: 判断是否需要重新计算 View 的大小，需要的话则计算
- layout: 判断是否需要重新计算 View 的位置，需要的话则计算
- draw: 判断是否需要重新绘制 View，需要的话则重绘制

### MeasureSpec

MeasureSpec表示的是一个32位的整形值，它的高2位表示测量模式SpecMode，低30位表示某种测量模式下的规格大小SpecSize。MeasureSpec 是 View 类的一个静态内部类，用来说明应该如何测量这个 View

| Mode        | 说明                                                         |
| ----------- | ------------------------------------------------------------ |
| UNSPECIFIED | 不指定测量模式, 父视图没有限制子视图的大小，子视图可以是想要的任何尺寸，通常用于系统内部，应用开发中很少用到。 |
| EXACTLY     | 精确测量模式，视图宽高指定为 match_parent 或具体数值时生效，表示父视图已经决定了子视图的精确大小，这种模式下 View 的测量值就是 SpecSize 的值 |
| AT_MOST     | 最大值测量模式，当视图的宽高指定为 wrap_content 时生效，此时子视图的尺寸可以是不超过父视图允许的最大尺寸的任何尺寸 |

### VelocityTracker

**VelocityTracker** 可用于追踪手指在滑动中的速度：

```java
view.setOnTouchListener(new View.OnTouchListener() {
    @Override
    public boolean onTouch(View v, MotionEvent event) {
        VelocityTracker velocityTracker = VelocityTracker.obtain();
        velocityTracker.addMovement(event);
        velocityTracker.computeCurrentVelocity(1000);
        int xVelocity = (int) velocityTracker.getXVelocity();
        int yVelocity = (int) velocityTracker.getYVelocity();
        velocityTracker.clear();
        velocityTracker.recycle();
        return false;
    }
});
```

### GestureDetector

**GestureDetector** 辅助检测用户的单击、滑动、长按、双击等行为



### View 的事件分发

主要包含3个方法：

- dispatchTouchEvent 
- onInterceptTouchEvent （只有ViewGroup才有）
- onTouchEvent 

首先是调用顶层ViewGroup 的dispatchTouchEvent 方法开始分发。以上三个方法关系的伪代码如下：

```java
public boolean dispatchTouchEvent(MotionEvent ev){
    boolean consume = false;
    if(onInterceptTouchEvent(ev)){
    	consume=onTouchEvent(ev);
    }else{
    	consume=child.dispatchTouchEvent(ev);
     }
    return consume;
}
```

```java
// 点击事件产生后，会直接调用dispatchTouchEvent（）方法
public boolean dispatchTouchEvent(MotionEvent ev) {   
    //代表是否消耗事件    
    boolean consume = false;    
    if (onInterceptTouchEvent(ev)) {   
        //如果onInterceptTouchEvent()返回true则代表当前View拦截了点击事件    
        //则该点击事件则会交给当前View进行处理   
        //即调用onTouchEvent (）方法去处理点击事件      
        consume = onTouchEvent (ev) ;    
    } else {      
        //如果onInterceptTouchEvent()返回false则代表当前View不拦截点击事件    
        //则该点击事件则会继续传递给它的子元素     
        //子元素的dispatchTouchEvent（）就会被调用，重复上述过程     
        //直到点击事件被最终处理为止      
        consume = child.dispatchTouchEvent (ev) ;   
    }   
    return consume;  
}
```



**requestDisallowInterceptTouchEvent**

该方法接受一个Boolean值，这个方法的作用是控制是否跳过所有父类的onInterceptTouchEvent.即是否禁止父类对事件进行拦截。

子View在onInterceptTouchEvent的ACTION_DOWN之后调用requestDisallowInterceptTouchEvent(true)，则此子View的所有父ViewGroup会跳过onInterceptTouchEvent回调，直接调用dispatchTransformedTouchEvent。

一般用来处理子类跳过父类Touch拦截，如：ScrollView嵌套ListView，ListView无法正常滚动，ScrollView默认拦截Touch事件，ListView无法收到Touch事件，无法正常滚动，那么就需要在ListView中重写onInterceptTouchEvent方法：

1.在down时调用getParent().requestDisallowInterceptTouchEvent(true),取消父类对事件的拦截

2.在up、cancel时调用getParent().requestDisallowInterceptTouchEvent(false)

## RecyclerView

RecyclerView的缓存分为四级

- **Scrap**

  对应ListView 的Active View，屏幕内的缓存数据，可以直接拿来复用。

  Scrap缓存列表（mChangedScrap、mAttachedScrap）是RecyclerView最先查找ViewHolder地方，它跟RecyclerViewPool或者ViewCache有很大的区别。

  mChangedScrap和mAttachedScrap只在布局阶段使用。其他时候它们是空的。布局完成之后，这两个缓存中的viewHolder，会移到mCacheView或者 RecyclerViewPool中。

  当LayoutManager开始布局的时候（预布局或者是最终布局），当前布局中的所有view，都会被dump到scrap中（具体实现可见LinearLayoutManager#onLayoutChildren()方法中调用了detachAndScrapAttachedViews() ），然后LayoutManager挨个地取回view，除非view发生了什么变化，否则它会马上从scrap中回到原来的位置。

  除了在布局时不为空外，还有另一个与scrap有关的规律：所有scrap的 view 都会跟RecyclerView分离。ViewGroup中的`attachView`和`detachView`方法跟addView和removeView方法很像，但是**不会触发请求布局会重绘的事件**。它们只是从ViewGroup的子view列表中删除对应的子view，并将该子view的parent设置为null。detached状态必须是临时，后面紧随着attach或者remove事件。

- **Cache**

  **最近**移出屏幕的缓存数据，**默认大小是2个**。当其容量被充满同时又有新的数据添加的时候，会根据FIFO原则，把优先进入的缓存数据移出并放到下一级缓存中，然后再把新的数据添加进来。Cache里面的数据是干净的，也就是携带了原来的ViewHolder的所有数据信息，数据可以直接来拿来复用。需要注意的是，**cache是根据position来寻找数据的**，这个postion是根据第一个或者最后一个可见的item的position以及用户操作行为（上拉还是下拉）。

  举个**栗子**：当前屏幕内第一个可见的item的position是1，用户进行了一个下拉操作，那么当前预测的position就相当于（1-1=0），也就是**position=0**的那个item要被拉回到屏幕，此时RecyclerView就从Cache里面找position=0的数据，如果找到了就直接拿来复用。

- **ViewCacheExtension**

  需要自定义，不明了，慎用

- **RecycledViewPool**

  Cache满了后，会格局FIFO原则，放入RecycledViewPool中，RecycledViewPool**默认的缓存数量是5个**。此处不同的是，放入RecycledViewPool前，**ViewHolder会被重置**。**RecycledViewPool是根据itemType**获取的，RecycledViewPool取出的Item需要重新进行数据绑定（`onBindViewHolder()`）。

一般情况下(忽略ViewCacheExtension，因为这个需要自己实现)，它有4个存放回收Holder的集合，分别是：

- 可直接重用的临时缓存：mAttachedScrap，mChangedScrap；
- 可直接重用的缓存：mCachedViews；
- 需重新绑定数据的缓存：mRecyclerPool.mScrap；

**为什么说前面两个是临时缓存呢？** 因为每当RecyclerView的dispatchLayout方法结束之前（当调用RecyclerView的reuqestLayout方法或者调用Adapter的一系列notify方法会回调这个dispatchLayout），它们里面的Holder都会移动到mCachedViews或mRecyclerPool.mScrap中。

**那为什么有两个呢？它们之间有什么区别吗？** 它们之间的区别就是：mChangedScrap只能在预布局状态下重用，因为它里面装的都是即将要放到mRecyclerPool中的Holder，而mAttachedScrap则可以在非预布局状态下重用。

**什么是预布局(PreLayout)？** 顾名思义，就是在真正布局之前，事先布局一次。但在预布局状态下，应该把已经remove掉的Item也layout出来，我们可以通过ViewHolder的LayoutParams.isViewRemoved()方法来判断这个ViewHolder是否已经被remove掉。 只有在Adapter的数据集更新时，并且调用的是除notifyDataSetChanged以外的一系列notify方法，预布局才会生效。这也是为什么调用notifyDataSetChanged方法不会播放Item动画的原因了。 这个其实有点像加载Bitmap的操作：先设置只读边，等获取到图片尺寸后设置好缩放比例再真正把图片加载进来。 要开启预布局的话，需要重写LayoutManager中的**supportsPredictiveItemAnimations**方法并**return true;** 这样就能生效了(当然，自带的那三个LayoutManager已经是开启了这个效果的)，当Adapter的数据集更新时，onLayoutChildren方法就会回调两次，第一次是预布局，第二次是真实的布局，我们也可以通过**state.isPreLayout()** 来判断当前是否为预布局状态，并根据这个状态来决定要layout的Item。

> 此处引用：https://blog.csdn.net/u011387817/article/details/81875021
>
> 笔记：https://github.com/JoneAcme/StudyNode/blob/master/android/RecyclerView.md



## Handler

### 机制

Handler 有两个主要用途：

1. 安排 Message 和 runnables 在将来的某个时刻执行; 
2. 将要在不同于自己的线程上执行的操作排入队列。(在多个线程并发更新UI的同时保证线程安全。)

Android 规定访问 UI 只能在主线程中进行，因为 Android 的 UI 控件不是线程安全的，多线程并发访问会导致 UI 控件处于不可预期的状态。为什么系统不对 UI 控件的访问加上锁机制？缺点有两个：加锁会让 UI 访问的逻辑变得复杂；其次锁机制会降低 UI 访问的效率。如果子线程访问 UI，那么程序就会抛出异常。ViewRootImpl 对UI操作做了验证，这个验证工作是由 ViewRootImpl的 `checkThread` 方法完成：

```java
ViewRootImpl.java
void checkThread() {
    if (mThread != Thread.currentThread()) {
        throw new CalledFromWrongThreadException(
                "Only the original thread that created a view hierarchy can touch its views.");
    }
}
```

- Message：Handler 接收和处理的消息对象
- MessageQueue：Message 的队列，先进先出，每一个线程最多可以拥有一个
- Looper：消息泵，是 MessageQueue 的管理者，会不断从 MessageQueue 中取出消息，并将消息分给对应的 Handler 处理，每个线程只有一个 Looper。

Handler 创建的时候会采用当前线程的 Looper 来构造消息循环系统，需要注意的是，线程默认是没有 Looper 的，直接使用 Handler 会报错，如果需要使用 Handler 就必须为线程创建 Looper，因为默认的 UI 主线程，也就是 ActivityThread，ActivityThread 被创建的时候就会初始化 Looper，这也是在主线程中默认可以使用 Handler 的原因。

### [屏障消息（同步屏障）](https://www.cnblogs.com/renhui/p/12875589.html)

Handler的Message种类分为3种：

- 普通消息
- 屏障消息
- 异步消息

其中普通消息又称为同步消息，屏障消息又称为同步屏障。

我们通常使用的都是普通消息，而屏障消息就是在消息队列中插入一个屏障，在屏障之后的所有普通消息都会被挡着，不能被处理。不过异步消息却例外，屏障不会挡住异步消息，因此可以这样认为：**屏障消息就是为了确保异步消息的优先级，设置了屏障后，只能处理其后的异步消息，同步消息会被挡住，除非撤销屏障。**

**`postSyncBarrier`**方法就是用来插入一个屏障到消息队列的

- 屏障消息和普通消息的区别在于屏障没有tartget，普通消息有target是因为它需要将消息分发给对应的target，而屏障不需要被分发，它就是用来挡住普通消息来保证异步消息优先处理的。
- 屏障和普通消息一样可以根据时间来插入到消息队列中的适当位置，并且只会挡住它后面的同步消息的分发。
- postSyncBarrier返回一个int类型的数值，通过这个数值可以撤销屏障。
- postSyncBarrier方法是私有的，如果我们想调用它就得使用反射。
- 插入普通消息会唤醒消息队列，但是插入屏障不会。

**屏障消息的工作原理**

```java
//MessageQueue
Message next() {
    //1、如果有消息被插入到消息队列或者超时时间到，就被唤醒，否则阻塞在这。
    nativePollOnce(ptr, nextPollTimeoutMillis);
    synchronized (this) {
        Message prevMsg = null;
        Message msg = mMessages;
        if (msg != null && msg.target == null) {//2、遇到屏障  msg.target == null
            do {
                prevMsg = msg;
                msg = msg.next;
            } while (msg != null && !msg.isAsynchronous());//3、遍历消息链表找到最近的一条异步消息
        }
        if (msg != null) {
            //4、如果找到异步消息
            if (now < msg.when) {//异步消息还没到处理时间，就在等会（超时时间）
                nextPollTimeoutMillis = (int) Math.min(msg.when - now, Integer.MAX_VALUE);
            } else {
                //异步消息到了处理时间，就从链表移除，返回它。
                mBlocked = false;
                if (prevMsg != null) {
                    prevMsg.next = msg.next;
                } else {
                    mMessages = msg.next;
                }
                msg.next = null;
                if (DEBUG) Log.v(TAG, "Returning message: " + msg);
                msg.markInUse();
                return msg;
            }
        } else {
            // 如果没有异步消息就一直休眠，等待被唤醒。
            nextPollTimeoutMillis = -1;
        }
        //...
    }
}
```

如果碰到屏障就遍历整个消息链表找到**最近的一条异步消息**，在遍历的过程中**只有异步消息才会被处理执行到** if (msg != null){}中的代码。

屏障消息就是通过这种方式就挡住了所有的普通消息。



### [子线程中怎么使用 Handler](https://mp.weixin.qq.com/s/2-MBLmMqUr_GARVEi81Ntg)

子线程中使用 Handler 需要先执行两个操作：Looper.prepare 和 Looper.loop。

- **Looper.prepare** 就是**创建了 Looper 并设置给 ThreadLocal**，这里的一个细节是**每个 Thread 只能有一个 Looper**，否则也会抛出异常。

- **Looper.loop** 就是开始读取 MessageQueue 中的消息，进行执行了。

> 主线程中为什么不用手动调用这两个方法呢?
>
> 答案：ActivityThread.main 中已经进行了调用



### [MessageQueue 如何等待消息](https://mp.weixin.qq.com/s/2-MBLmMqUr_GARVEi81Ntg)

通过 Looper.loop 方法，是 MessageQueue.next() 来获取消息的，如果没有消息，那就会阻塞在这里

在 native 侧，最终是使用了 epoll_wait 来进行等待的。涉及**Linux 内核的epoll机制**，挂起进程，放入等待队列，等待唤醒

。期间CPU不会轮训此进程，也就是此进程不占用CPU。

###  [为什么不用 wait 而用 epoll 呢？](https://mp.weixin.qq.com/s/2-MBLmMqUr_GARVEi81Ntg)

 java 中的 wait / notify 也能实现阻塞等待消息的功能，在 Android 2.2 及以前，也确实是这样做的。

需要处理 native 侧的事件，所以只使用 java 的 wait / notify 就不够用了。

不过这里最开始使用的还是 select，后面才改成 epoll。

- select 无法定位是哪个socket唤醒的，需要遍历两次定位socket
- epoll 可高效地监视多个 socket。内核维护一个“就绪列表”，引用收到数据的 socket，就能避免遍历。

[select、poll、epoll](https://my.oschina.net/editorial-story/blog/3052308)

### [多个线程给 MessageQueue 发消息，如何保证线程安全](https://my.oschina.net/editorial-story/blog/3052308)

MessageQueue加锁

```java
// MessageQueue.java
boolean enqueueMessage(Message msg, long when) {
    synchronized (this) {
        // ...
    }
}
```



### Handler [消息延迟是怎么处理的](https://my.oschina.net/editorial-story/blog/3052308)

Handler post 消息以后，一直调用到 MessageQueue.enqueueMessage 里，其中最重要的一步操作就是传入的时间是 uptimeMillis + delayMillis。

post 一个延迟消息时，在 MessageQueue 中会根据 when 的时长进行一个顺序排序。

1. 将我们传入的延迟时间转化成距离开机时间的毫秒数（不适用Systime.currenttimemillis的原因是时间可更改）

2. MessageQueue 中根据上一步转化的时间进行顺序排序

3. 在 MessageQueue.next 获取消息时，对比当前时间（now）和第一步转化的时间（when），如果 now < when，则通过 epoll_wait 的 timeout 进行等待

4. 如果该消息需要等待，会进行 idel handlers 的执行，执行完以后会再去检查此消息是否可以执行

   

### [View.post 和 Handler.post 的区别](https://mp.weixin.qq.com/s/2-MBLmMqUr_GARVEi81Ntg)

View.post 都是通过 ViewRootImpl 内部的 Handler 进行处理的。

1. 如果在 performTraversals 前调用 View.post，则会将消息进行保存，之后在 dispatchAttachedToWindow 的时候通过 ViewRootImpl 中的 Handler 进行调用。
2. 如果在 performTraversals 以后调用 View.post，则直接通过 ViewRootImpl 中的 Handler 进行调用。



###  View.post 里可以拿到 View 的宽高信息

View.post 的 Runnable 执行的时候，已经执行过 performTraversals 了，也就是 View 的 measure layout draw 方法都执行过了，自然可以获取到 View 的宽高信息了。



###  非 UI 线程真的不能操作 View 吗

检查，其实并不是检查主线程，是检查 mThread != Thread.currentThread，而 mThread 指的是 ViewRootImpl 创建的线程。

所以非 UI 线程确实不能操作 View，但是检查的是创建的线程是否是当前线程，因为 ViewRootImpl 创建是在主线程创建的，所以在非主线程操作 UI 过不了这里的检查。



### 内存泄漏

**场景一**

```java
public class MainActivity extends AppCompatActivity {
    private CalendarPageView calendar;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        new Handler().postDelayed(new Runnable() {
            @Override
            public void run() {
                
            }
        },10000);
    }
}
```

在Java中在内部创建对象之后会隐式的持有外部对象，也就是说new Handler()之后Handler对象对Activity就有了一个持有，那么此时finish掉Activity的话是没办法回收的。这就造成了内存泄漏。

**场景二**

```java
private Handler handler = new Handler(){
        @Override
        public void dispatchMessage(Message msg) {
            super.dispatchMessage(msg);
            Activity activity = MainActivity.this;
        }
    };
```

在里面Handler不仅仅是隐式的持有，也有显示的持有Activity，若执行延时操作也会导致Activity无法被回收。 对于隐式持有，Handler的postDelayed方法，实际上是先把Message添加到MessageQueue中，进而等待Message的执行，所以而Message又是在主线程的Looper中穿件，所以持有链接是ActivityThread->Looper->MessageQueue->Message->Handler->Activity

**解决方案**

1. Handler的具体实现使用静态内部类的方式（静态内部类不会持有外部类的引用）
2. 再对外部的Activity进行弱引用。
3. 若是Handler有延时操作，那么Handler依然会存在，如果延时消息在Activity销毁之后没有存在的必要，那么我们可以在Activity销毁之后删除Handler的消息。
   代码：

```java
  @Override
    protected void onDestroy() {
        super.onDestroy();
        handler.removeCallbacksAndMessages(null);
    
```



## [事件分发](https://mp.weixin.qq.com/s/N45bKKtggkBqtAr6d5eMUw)

#### 分发步骤

 Activity -> Window -> DecorView -> ViewGroup -> View 的 dispatchTouchEvent 

- dispatchTouchEvent
- onInterceptTouchEvent 
- onTouchEvent

```java
// 3个方法的伪代码实现
// ViewGroup 
dispatchTouchEvent(){
    if(!onInterceptTouchEvent()){
        if(child instanceof ViewGroup)
        	child.dispatchTouchEvent()
        else 
            child.onTouchEvent()
    }
}
```

#### CANCEL 事件什么时候会触发？

- View 收到 ACTION_DOWN 事件以后，上一个事件还没有结束（可能因为 APP 的切换、ANR 等导致系统扔掉了后续的事件），这个时候会先执行一次 ACTION_CANCEL

- 子 View 之前拦截了事件，但是后面父 View 重新拦截了事件，这个时候会给子 View 发送 ACTION_CANCEL 事件

  

#### 如何解决滑动冲突

1. 通过重写父类的 onInterceptTouchEvent 来拦截滑动事件
2. 通过在子类中调用 parent.requestDisallowInterceptTouchEvent 来通知父类是否要拦截事件，requestDisallowInterceptTouchEvent 会设置 FLAG_DISALLOW_INTERCEPT 标志，这个在最开始的伪代码那里做过介绍



## 动画

### 补间动画

- 平移动画（`Translate`）
- 缩放动画（`scale`）
- 旋转动画（`rotate`）
- 透明度动画（`alpha`）

**作用对象**

视图控件（`View`）

> 1. 如`Android`的`TextView、Button`等等
> 2. 不可作用于`View`组件的属性，如：颜色、背景、长度等等

**使用场景**

- 补间动画常用于视图View的一些标准动画效果：平移、旋转、缩放 & 透明度；
- 特殊场景：`Activity` 的切换效果、`Fragement` 的切换效果、视图组（`ViewGroup`）中子元素的出场效果



### 帧动画

**作用对象**

视图控件（`View`）

> 1. 如`Android`的`TextView、Button`等等
> 2. 不可作用于`View`组件的属性，如：颜色、背景、长度等等

**特点**

- 优点：使用简单、方便
- 缺点：容易引起 `OOM`，因为会使用大量 & 尺寸较大的图片资源

> 尽量避免使用尺寸较大的图片

**应用场景**

较为复杂的个性化动画效果。

> 使用时一定要避免使用尺寸较大的图片，否则会引起OOM



### 属性动画

**作用对象**

任意Java对象，不局限于视图控件

原理：在一定时间间隔内，通过不断对值进行改变&不断对值赋值给对象的属性，实现该对象属性上的动画效果。可以是任意对象的任意属性。

**特点**

- 不再仅限于View对象
- 动画效果丰富

![](https://imgconvert.csdnimg.cn/aHR0cDovL3VwbG9hZC1pbWFnZXMuamlhbnNodS5pby91cGxvYWRfaW1hZ2VzLzk0NDM2NS1lNzVlMzI4ZjdlM2ZhYjU5LnBuZz9pbWFnZU1vZ3IyL2F1dG8tb3JpZW50L3N0cmlwJTdDaW1hZ2VWaWV3Mi8yL3cvMTI0MA)

**估值器（TypeEvaluator）**

作用：设置动画 初始值 过渡到 结束值 具体数值

**插值器（Interpolator）**

作用：设置 属性值 从初始值过渡到结束值 的变化规律

### 二者区别

**两类动画的根本区别在于：是否改变动画本身的属性**：

- 视图动画：无改变动画的属性
  因为视图动画在动画过程中仅对图像进行变换，从而达到了动画效果

> 变换操作包括：平移、缩放、旋转和透明

- 属性动画：改变了动画属性
  因属性动画在动画过程中对动态改变了对象属性，从而达到了动画效果
- 特别注意
  使用视图动画时：无论动画结果在哪，该View的位置不变 & 响应区域都是在原地，不会根据结果而移动；
  而属性动画 则会通过改变属性 从而使动画移动

## HandlerThread

一个包含`Looper`的Thread，通过这个Looper可以创建Handler

内部原理 = `Thread`类 + `Handler`类机制，即：

- 通过继承`Thread`类，**快速地创建1个带有Looper对象的新工作线程**
- 通过封装`Handler`类，**快速创建Handler& 与其他线程进行通信**



**优点**：只要开启一个线程，就可以处理多个耗时任务。

**缺点**：任务是串行执行的，不能并行执行。一旦队列中有某个任务执行时间过长，就会导致后续的任务都会被延迟处理。

**注意事项**

1. HandlerThread不再需要使用的时候，要调用`quitSafe()`或者`quit()`方法来结束线程。
2. `quitSafe()`会等待正在处理的消息处理完后再退出，而`quit()`不管是否正在处理消息，直接移除所有回调。



## IntentService 

IntentService 可用于执行后台耗时的任务，当**任务执行后会自动停止**，由于其是 Service 的原因，它的优先级比单纯的线程要高，所以 IntentService 适合执行一些高优先级的后台任务。

定义：继承四大组件之一的`Service`。

作用：处理异步请求 & 实现多线程

使用场景：线程任务 需 **按顺序**、**在后台执行**。如：离线下载

![img](https:////upload-images.jianshu.io/upload_images/944365-fa5bfe6dffa531ce.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/670)

工作流程

特别注意：若启动`IntentService`多次，那么 每个耗时操作 则 **以队列的方式**在 `IntentService`的 `onHandleIntent`回调方法中依次执行，执行完自动结束

接下来，我们将通过 源码分析 解决以下问题：

- `IntentService`如何单独开启1个新的工作线程
- `IntentService`如何通过`onStartCommand()`将Intent 传递给服务 & 依次插入到工作队列中

总结

从上面源码可看出：`IntentService`本质 = `Handler`+ `HandlerThread`：

1. 通过`HandlerThread`单独开启1个工作线程：`IntentService`
2. 创建1个内部 `Handler`：`ServiceHandler`
3. 绑定 `ServiceHandler`与 `IntentService`
4. 通过 `onStartCommand()`传递服务`intent`到`ServiceHandler`、依次插入`Intent`到工作队列中 & 逐个发送给 `onHandleIntent()`
5. 通过`onHandleIntent()`依次处理所有`Intent`对象所对应的任务

> 因此通过复写`onHandleIntent()`& 在里面 根据`Intent`的不同进行不同线程操作 即可

------

注意事项

1. **工作任务队列 = 顺序执行**

> 即 若一个任务正在`IntentService`中执行，此时再发送1个新的任务请求，这个新的任务会一直等待直到前面一个任务执行完毕后才开始执行

- 原因：

  1. 由于`onCreate()`只会调用一次 = 只会创建1个工作线程；

  2. 当多次调用 `startService(Intent)`时（即 `onStartCommand（）`也会调用多次），其实不会创建新的工作线程，只是把消息加入消息队列中 & 等待执行。
     **所以，多次启动 IntentService 会按顺序执行事件**

> 若服务停止，则会清除消息队列中的消息，后续的事件不执行

2. **不建议通过 bindService() 启动 IntentService**

原因：

```java
// 在IntentService中，onBind()`默认返回null
@Override
public IBinder onBind(Intent intent) {
    return null;
}
```

- 采用 `bindService()`启动 `IntentService`的生命周期如下：

> onCreate() ->> onBind() ->> onunbind()->> onDestory()

- 即，并不会调用`onStart()`或 `onStartcommand()`，**故不会将消息发送到消息队列，那么onHandleIntent()将不会回调，即无法实现多线程的操作**

> 此时，你应该使用`Service`，而不是`IntentService`

------

 **IntentService与Service的区别**

![img](https:////upload-images.jianshu.io/upload_images/944365-c85aac01a4325ed9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000)

 **IntentService与其他线程的区别**

![img](https:////upload-images.jianshu.io/upload_images/944365-6a86410410a8278b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/730)



## Window / WindowManager

Window 是一个抽象类，它的具体实现是 PhoneWindow。WindowManager 是外界访问 Window 的入口，Window 的具体实现位于 WindowManagerService 中，WindowManager 和 WindowManagerService 的交互是一个 IPC 过程。Android 中所有的视图都是通过 Window 来呈现，因此 Window 实际是 View 的直接管理者。

| Window 类型        | 说明                                               | 层级      |
| ------------------ | -------------------------------------------------- | --------- |
| Application Window | 对应着一个 Activity                                | 1~99      |
| Sub Window         | 不能单独存在，只能附属在父 Window 中，如 Dialog 等 | 1000~1999 |
| System Window      | 需要权限声明，如 Toast 和 系统状态栏等             | 2000~2999 |

Window 是一个抽象的概念，每一个 Window 对应着一个 View 和一个 ViewRootImpl。Window 实际是不存在的，它是以 View 的形式存在。对 Window 的访问必须通过 WindowManager，WindowManager 的实现类是 WindowManagerImpl。而具体操作是由WindowManagerGlobal 处理。

**添加一个Window**

```kotlin
 val floatButton = Button(this)
        floatButton.text = "button"
        val layoutparams = WindowManager.LayoutParams(
            WindowManager.LayoutParams.WRAP_CONTENT,
            WindowManager.LayoutParams.WRAP_CONTENT,
            0,
            0,
            PixelFormat.TRANSLUCENT
        )
        layoutparams.flags = WindowManager.LayoutParams.FLAG_NOT_TOUCH_MODAL and WindowManager.LayoutParams.FLAG_NOT_FOCUSABLE and WindowManager.LayoutParams.FLAG_ALLOW_LOCK_WHILE_SCREEN_ON
        layoutparams.gravity = Gravity.LEFT and Gravity.LEFT
        layoutparams.x = 300
        layoutparams.y = 100
        windowManager.addView(floatButton,layoutparams)
```

如果想要删除一个Window，只需要删除它里面的View即可

## Bitmap

**Bitmap 中有两个内部枚举类：**

- Config 是用来设置颜色配置信息
- CompressFormat 是用来设置压缩方式

| Config                  | 单位像素所占字节数 | 解析                                                         |
| ----------------------- | ------------------ | ------------------------------------------------------------ |
| Bitmap.Config.ALPHA_8   | 1                  | 颜色信息只由透明度组成，占8位                                |
| Bitmap.Config.ARGB_4444 | 2                  | 颜色信息由rgba四部分组成，每个部分都占4位，总共占16位        |
| Bitmap.Config.ARGB_8888 | 4                  | 颜色信息由rgba四部分组成，每个部分都占8位，总共占32位。是Bitmap默认的颜色配置信息，也是最占空间的一种配置 |
| Bitmap.Config.RGB_565   | 2                  | 颜色信息由rgb三部分组成，R占5位，G占6位，B占5位，总共占16位  |
| RGBA_F16                | 8                  | Android 8.0 新增（更丰富的色彩表现HDR）                      |
| HARDWARE                | Special            | Android 8.0 新增 （Bitmap直接存储在graphic memory）          |

> 通常优化 Bitmap 时，当需要做性能优化或者防止 OOM，我们通常会使用 Bitmap.Config.RGB_565 这个配置，因为 Bitmap.Config.ALPHA_8 只有透明度，显示一般图片没有意义，Bitmap.Config.ARGB_4444 显示图片不清楚， Bitmap.Config.ARGB_8888 占用内存最多。

| CompressFormat             | 解析                                                         |
| -------------------------- | ------------------------------------------------------------ |
| Bitmap.CompressFormat.JPEG | 表示以 JPEG 压缩算法进行图像压缩，压缩后的格式可以是 `.jpg` 或者 `.jpeg`，是一种有损压缩 |
| Bitmap.CompressFormat.PNG  | 颜色信息由 rgba 四部分组成，每个部分都占 4 位，总共占 16 位  |
| Bitmap.Config.ARGB_8888    | 颜色信息由 rgba 四部分组成，每个部分都占 8 位，总共占 32 位。是 Bitmap 默认的颜色配置信息，也是最占空间的一种配置 |
| Bitmap.Config.RGB_565      | 颜色信息由 rgb 三部分组成，R 占 5 位，G 占 6 位，B 占 5 位，总共占 16 位 |



## Android中的多进程

方法：

1. 指定四大组件的android:process属性
2. 通过JNI在native层fork一个新的进程

android:process属性取值：

- xxx.xxx.xxx.new

- :new

  “:”的含义：

  1. 指要在当前的进程名前面附加上包名
  2. 以“:”开头的进程属于当前应用的私有线程，其他应用不能与它跑在同一进程中
  3. 而不以“:”开头的进程属于全局线程，其他应用通过ShareUID的方式可以和它跑在同一进程中

多进程会造成的问题：

- 静态成员和单例模式完全失效
- 线程同步机制完全失效
- SharedPerences可靠性下降
- Application会多次创建



## LayoutInflater 过程

获取LayoutInflater：

1. Activity.getLayoutInflater()；
2. Context.getSystemService(Context.LAYOUT_INFLATER_SERVICE) ;
3. LayoutInflater.from(context);

最终都是通过Context.getSystemService(Context.LAYOUT_INFLATER_SERVICE) ;

解析XML的三种方式：DOM、SAX、PULL

|          | 特点                                                         |
| -------- | ------------------------------------------------------------ |
| DOM解析  | 整个文档树在内存中、操作方便，占用内存大                     |
| SAX解析  | 事件驱动、顺序解析、解析效率高，占用内存少；使用回调函数来实现。 缺点是不能倒退。 |
| PULL解析 | android内置的解析器，与SAX类似；Pull解析器并没有强制要求提供触发的方法 |

过程：

1. 通过 XML 的 Pull 解析方式获取 View 的标签；
2. 通过标签以反射的方式来创建 View 对象；
3. 如果是ViewGroup的话则会对子 View 遍历并重复以上步骤，然后 add 到父 View 中；

**与之相关的几个方法：inflate ——》 rInflate ——》 createViewFromTag ——》 createView ；**





## IPC相关

### 序列化

Serializable：Java序列化接口，写入文件保存、传输

Parcelable：Android序列号接口，写入内存直接传输

**Parcelable 与 Serializable 对比**

- Serializable 使用 I/O 读写存储在硬盘上，而 Parcelable 是直接在内存中读写
- Serializable 会使用反射，序列化和反序列化过程需要大量 I/O 操作， Parcelable 自已实现封送和解封（marshalled &unmarshalled）操作不需要用反射，数据也存放在 Native 内存中，效率要快很多

### IPC方式

| 名称            | 优点                                                         | 缺点                                                         | 适用场景                                                     |
| --------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Bundle          | 简单易用                                                     | 只能传输 Bundle 支持的数据类型                               | 四大组件间的进程间通信                                       |
| 文件共享        | 简单易用                                                     | 不适合高并发场景，并且无法做到进程间即时通信                 | 无并发访问情形，交换简单的数据实时性不高的场景               |
| AIDL            | 功能强大，支持一对多并发通信，支持实时通信                   | 使用稍复杂，需要处理好线程同步                               | 一对多通信且有 RPC 需求                                      |
| Messenger       | 功能一般，支持一对多串行通信，支持实时通信                   | 不能很处理高并发清醒，不支持 RPC，数据通过 Message 进行传输，因此只能传输 Bundle 支持的数据类型 | 低并发的一对多即时通信，无RPC需求，或者无需返回结果的RPC需求 |
| ContentProvider | 在数据源访问方面功能强大，支持一对多并发数据共享，可通过 Call 方法扩展其他操作 | 可以理解为受约束的 AIDL，主要提供数据源的 CRUD 操作          | 一对多的进程间数据共享                                       |
| Socket          | 可以通过网络传输字节流，支持一对多并发实时通信               | 实现细节稍微有点烦琐，不支持直接的RPC                        | 网络数据交换                                                 |

### Binder

[Binder解析](https://juejin.im/post/6890088205916307469)

Binder 是 Android 中的一个类，实现了 IBinder 接口。从 IPC 角度来说，Binder 是 Android 中的一种扩进程通信方方式。从 Android 应用层来说，Binder 是客户端和服务器端进行通信的媒介，当 bindService 的时候，服务端会返回一个包含了服务端业务调用的 Binder 对象。

Binder 相较于传统 IPC 来说更适合于Android系统，具体原因的包括如下三点：

- Binder 本身是 C/S 架构的，这一点更符合 Android 系统的架构
- 性能上更有优势：管道，消息队列，Socket 的通讯都需要两次数据拷贝，而 Binder 只需要一次。要知道，对于系统底层的 IPC 形式，少一次数据拷贝，对整体性能的影响是非常之大的
- 安全性更好：传统 IPC 形式，无法得到对方的身份标识（UID/GID)，而在使用 Binder IPC 时，这些身份标示是跟随调用过程而自动传递的。Server 端很容易就可以知道 Client 端的身份，非常便于做安全检查

示例：

**新建AIDL接口文件**

```java
RemoteService.aidl
package com.example.mystudyapplication3;

interface IRemoteService {

    int getUserId();

}
```

系统会自动生成 `IRemoteService.java`:

```java
/*
 * This file is auto-generated.  DO NOT MODIFY.
 */
package com.example.mystudyapplication3;
// Declare any non-default types here with import statements
//import com.example.mystudyapplication3.IUserBean;

public interface IRemoteService extends android.os.IInterface {
    /**
     * Local-side IPC implementation stub class.
     */
    public static abstract class Stub extends android.os.Binder implements com.example.mystudyapplication3.IRemoteService {
        private static final java.lang.String DESCRIPTOR = "com.example.mystudyapplication3.IRemoteService";

        /**
         * Construct the stub at attach it to the interface.
         */
        public Stub() {
            this.attachInterface(this, DESCRIPTOR);
        }

        /**
         * Cast an IBinder object into an com.example.mystudyapplication3.IRemoteService interface,
         * generating a proxy if needed.
         */
        public static com.example.mystudyapplication3.IRemoteService asInterface(android.os.IBinder obj) {
            if ((obj == null)) {
                return null;
            }
            android.os.IInterface iin = obj.queryLocalInterface(DESCRIPTOR);
            if (((iin != null) && (iin instanceof com.example.mystudyapplication3.IRemoteService))) {
                return ((com.example.mystudyapplication3.IRemoteService) iin);
            }
            return new com.example.mystudyapplication3.IRemoteService.Stub.Proxy(obj);
        }

        @Override
        public android.os.IBinder asBinder() {
            return this;
        }

        @Override
        public boolean onTransact(int code, android.os.Parcel data, android.os.Parcel reply, int flags) throws android.os.RemoteException {
            java.lang.String descriptor = DESCRIPTOR;
            switch (code) {
                case INTERFACE_TRANSACTION: {
                    reply.writeString(descriptor);
                    return true;
                }
                case TRANSACTION_getUserId: {
                    data.enforceInterface(descriptor);
                    int _result = this.getUserId();
                    reply.writeNoException();
                    reply.writeInt(_result);
                    return true;
                }
                default: {
                    return super.onTransact(code, data, reply, flags);
                }
            }
        }

        private static class Proxy implements com.example.mystudyapplication3.IRemoteService {
            private android.os.IBinder mRemote;

            Proxy(android.os.IBinder remote) {
                mRemote = remote;
            }

            @Override
            public android.os.IBinder asBinder() {
                return mRemote;
            }

            public java.lang.String getInterfaceDescriptor() {
                return DESCRIPTOR;
            }

            @Override
            public int getUserId() throws android.os.RemoteException {
                android.os.Parcel _data = android.os.Parcel.obtain();
                android.os.Parcel _reply = android.os.Parcel.obtain();
                int _result;
                try {
                    _data.writeInterfaceToken(DESCRIPTOR);
                    mRemote.transact(Stub.TRANSACTION_getUserId, _data, _reply, 0);
                    _reply.readException();
                    _result = _reply.readInt();
                } finally {
                    _reply.recycle();
                    _data.recycle();
                }
                return _result;
            }
        }

        static final int TRANSACTION_getUserId = (android.os.IBinder.FIRST_CALL_TRANSACTION + 0);
    }

    public int getUserId() throws android.os.RemoteException;
}
```

| 方法                     | 含义                                                         |
| ------------------------ | ------------------------------------------------------------ |
| DESCRIPTOR               | Binder 的唯一标识，一般用当前的 Binder 的类名表示            |
| asInterface(IBinder obj) | 将服务端的 Binder 对象成客户端所需的 AIDL 接口类型对象，这种转换过程是区分进程的，如果位于同一进程，返回的就是 Stub 对象本身，否则返回的是系统封装后的 Stub.proxy 对象。 |
| asBinder                 | 用于返回当前 Binder 对象                                     |
| onTransact               | 运行在服务端中的 Binder 线程池中，远程请求会通过系统底层封装后交由此方法来处理 |

| 定向 tag | 含义                                                         |
| -------- | ------------------------------------------------------------ |
| in       | 数据只能由客户端流向服务端，服务端将会收到客户端对象的完整数据，客户端对象不会因为服务端对传参的修改而发生变动。 |
| out      | 数据只能由服务端流向客户端，服务端将会收到客户端对象，该对象不为空，但是它里面的字段为空，但是在服务端对该对象作任何修改之后客户端的传参对象都会同步改动。 |
| inout    | 服务端将会接收到客户端传来对象的完整信息，并且客户端将会同步服务端对该对象的任何变动。 |

**流程**

[![img](https://camo.githubusercontent.com/3eda644a5fd0cfc206daa5c6bce57c3e38a1802d/687474703a2f2f6769747975616e2e636f6d2f696d616765732f62696e6465722f62696e6465725f73746172745f736572766963652f62696e6465725f6970635f617263682e6a7067)](https://camo.githubusercontent.com/3eda644a5fd0cfc206daa5c6bce57c3e38a1802d/687474703a2f2f6769747975616e2e636f6d2f696d616765732f62696e6465722f62696e6465725f73746172745f736572766963652f62696e6465725f6970635f617263682e6a7067)

### AIDL 通信

Android Interface Definition Language

使用示例：

- **新建AIDL接口文件**

```java
// RemoteService.aidl
package com.example.mystudyapplication3;

interface IRemoteService {

    int getUserId();

}
```

- **创建远程服务**

```java
public class RemoteService extends Service {

    private int mId = -1;

    private Binder binder = new IRemoteService.Stub() {

        @Override
        public int getUserId() throws RemoteException {
            return mId;
        }
    };

    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        mId = 1256;
        return binder;
    }
}
```

- **声明远程服务**

```xml
<service
    android:name=".RemoteService"
    android:process=":aidl" />
```

- **绑定远程服务**

```java
public class MainActivity extends AppCompatActivity {

    public static final String TAG = "wzq";

    IRemoteService iRemoteService;
    private ServiceConnection mConnection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            iRemoteService = IRemoteService.Stub.asInterface(service);
            try {
                Log.d(TAG, String.valueOf(iRemoteService.getUserId()));
            } catch (RemoteException e) {
                e.printStackTrace();
            }
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {
            iRemoteService = null;
        }
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        bindService(new Intent(MainActivity.this, RemoteService.class), mConnection, Context.BIND_AUTO_CREATE);
    }
}
```

### Messenger

Messenger可以在不同进程中传递 Message 对象，在Message中放入我们需要传递的数据，就可以轻松地实现数据的进程间传递了。Messenger 是一种轻量级的 IPC 方案，底层实现是 AIDL。



## Android中的虚拟机

android 5.0之前使用dalvik 虚拟机，之后采用art 虚拟机

### Dalvik

- Dalvik使用JIT
- 使用.dex字节码，是针对Android设备优化后的DVM所使用的运行时编译字节码
- .odex是对dex的优化，deodex在系统第一次开机时会提取所有apk内的dex文件，odex优化将dex提前提取出，加快了开机的速度和程序运行的速度

### ART

- ART 使用AOT
- 在安装apk时会进行预编译，生成OAT文件，仍以.odex保存，但是与Dalvik下不同，这个文件是可执行文件
- dex、odex 均可通过dex2oat生成oat文件，以实现兼容性
- 在大型应用安装时需要更多时间和空间

### Android N引入的混合编译

在Android N中引入了一种新的编译模式，同时使用JIT和AOT。



## Android 系统服务创建、获取

**获取过程：**

1. 应用程序进程调用**getSystemService**
2. 通过**binder跨进程通信**，拿到**ServiceManager进程的BpServiceManager**对象
3. 通过**BpServiceManager获取到系统服务**



**注册系统服务**：

系统服务可以分成两大类：

1. 有**独立进程**的ServiceManager、SurfaceFlinger等，他们在**init进程启动时就会被fork创建**；

2. 是**非独立进程**的AMS、PMS、WMS等，他们在init进程fork出Zygote进程，Zygote进程fork出的**SystemServer进程创建**。

​		不管是**由init进程启动的独立进程的系统服务**如SurfaceFlinger，还是**由SystemServer进程启动的非独立进程的系统服务**如AMS，都是在**ServiceManager进程**中完成注册和获取的，在跨进程通信上使用了Android的binder机制。

[详细资料](https://juejin.im/post/6885640823484973064#heading-2)

## 线上日志收集、问题排查

### 微信Mars-xlog

mars 是微信官方的终端基础组件，是一个使用 C++ 编写的业务性无关，平台性无关的基础组件。目

- 方案1.日志加密写入磁盘

  缺点：产生大量IO导致性能问题

- 方案2.写到缓存，一定大小再写入文件

  优点：可由C++实现一个平台性无关的日志模块，避免Android下频繁 GC 的问题

  缺点：程序 crash 时，异常退出，不一定能捕获到日志，丢日志

- **mmap方案**：使用逻辑内存对磁盘文件进行映射

  优点：

  1. 中间只是进行映射没有任何拷贝操作，避免了写文件的数据拷贝。
  2. mmap几乎和直接写内存一样的性能，而且 mmap 既不会丢日志

  回写时机：

  - 内存不足
  - 进程 crash
  - 调用 msync 或者 munmap
  - 不设置 MAP_NOSYNC 情况下 30s-60s(仅限FreeBSD)

  缺点：集中压缩导致 CPU 短时间飙高，以及压缩前容易被窥探，如果这个压缩单位中有数据损坏，那么后面的日志也都解压不出来。

- ##### xlog 方案总结

  1. 由C++实现一个平台性无关的日志模块，避免Android下频繁 GC 的问题

  2. 使用逻辑内存对磁盘文件进行映射，避免了写文件的数据拷贝

  3. 使用流式方式对单行日志进行压缩，压缩加密后写进作为 log 中间 buffer的 mmap 中

     

  优点：在mmap的基础上，使用流式压缩兼顾**流畅性 完整性 容错性**，除非 IO 损坏或者磁盘没有可用空间，基本可以保证不会丢失任何一行日志。

  其他策略：

  - 每次启动的时候会清理日志，防止占用太多用户磁盘空间
  - 为了防止 sdcard 被拔掉导致写不了日志，支持设置缓存目录，当 sdcard 插上时会把缓存目录里的日志写入到 sdcard 上
  - ……

### 美团Logan

实现逻辑与微信xlog 相同

1. 使用流式的加密和压缩，避免了CPU峰值，同时减少了CPU使用
2. 跨平台C库提供了日志协议数据的格式化处理，针对大日志的分片处理，引入了MMAP机制解决了日志丢失问题
3. 使用AES进行日志加密确保日志安全性
4. 提供了主动上报接口。

**Logan日志回捞**

1. 后台回捞，依赖于Push透传。客户端被唤醒接收Push消息，受到一些条件影响：
   - Android想要后台唤醒App，需要确保Push进程在后台存活；
   - iOS想要后台唤醒APP，需要确保用户开启后台刷新开关；
   - 网络环境太差，Android上Push长连建立不成功。

2. 用户主动上报

   用户投诉->指导用户上传日志->查看、分析用户日志





## [AOP思想](https://juejin.im/post/6844904017760370695)

从织入的时机的角度看，可以分为源码阶段、class阶段、dex阶段、运行时织入。

**源码阶段、class阶段、dex织入**，由于他们都发生在class加载到虚拟机前，我们统称为静态织入， 而在运行阶段发生的改动，我们统称为动态织入。

常见的技术框架如下表：

| 织入时机 | 技术框架                      |
| :------: | ----------------------------- |
| 静态织入 | APT，AspectJ、ASM、Javassit   |
| 动态织入 | java动态代理，cglib、Javassit |

静态织入发生在编译器，因此几乎不会对运行时的效率产生影响；动态织入发生在运行期，可直接将字节码写入内存，并通过反射完成类的加载，所以效率相对较低，但更灵活。

动态织入的前提是类还未被加载，你不能将一个已经加载的类经过修改再次加载，这是ClassLoader的限制。但是可以通过另一个ClassLoader进行加载，虚拟机允许两个相同类名的class被不同的ClassLoader加载，在运行时也会被认为是两个不同的类，因此需要注意不能相互赋值， 不然会抛出ClassCastException。



### APT

**APT**（Annotation Processing Tool）即注解处理器，在Gradle 版本>=2.2后被annotationProcessor取代。

它用来在编译时扫描和处理注解，扫描过程可使用[ auto-service ](https://github.com/google/auto/tree/master/service)来简化寻找注解的配置，在处理过程中可生成java文件（创建java文件通常依赖[ javapoet ](https://github.com/square/javapoet)这个库）。常用于生成一些模板代码或运行时依赖的类文件，比如常见的ButterKnife、Dagger、ARouter，它的优点是简单方便。

APT技术的不足，通常只是用来创建新的类，而不能对原有类进行改动，在不能改动的情况下，只能通过反射实现动态化。

### AspectJ

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

### ASM

ASM是一个字节码操作框架，可用来动态生成字节码或者对现有的类进行增强。ASM可以直接生成二进制的class字节码，也可以在class被加载进虚拟机前动态改变其行为，比如方法执行前后插入代码，添加成员变量，修改父类，添加接口等等。

> Matrix是微信开源的一个APM框架，其中TraceCanary子模块用于监测帧率低、卡顿、ANR等场景，具备函数耗时统计的功能。
>
> 为了实现函数的耗时统计，通常的做法都是在函数执行开始和结束为止进行插桩，最后以两个插桩点的时间差为函数的执行时间。

- 

**ASM优点：**

- 由于直接操作的是字节码，因此相比其他框架效率更高。
- 从ASM5开始已经支持Java8的部分语法，比如lamabda表达式。
- 因为ASM偏向底层，很多其他的上层框架也以ASM作为其底层操作字节码的技术栈，比如Groovy、cglib。

**ASM不足：**

1. 过滤类和方法需要硬编码，且不够灵活，需要对插件进行二次封装，而在AspectJ中已经封装好了切面表达式。
2. 很难实现在方法调用前后织入新的代码，而在AspectJ中一个call关键字就解决了。
3. 如果需要在指定的类，指定的方法中织入代码，需要编写相应的过滤条件，这也是相比于AspectJ而言不太方便的地方，AspectJ可通过声明切面注解完成精准的织入。



[**Matrix for Android**](https://github.com/Tencent/matrix#matrix_android_cn)

Matrix-android 当前监控范围包括：应用安装包大小，帧率变化，启动耗时，卡顿，慢方法，SQLite 操作优化，文件读写，内存泄漏等等。

- APK Checker: 针对 APK 安装包的分析检测工具，根据一系列设定好的规则，检测 APK 是否存在特定的问题，并输出较为详细的检测结果报告，用于分析排查问题以及版本追踪
- Resource Canary: 基于 WeakReference 的特性和 [Square Haha](https://github.com/square/haha) 库开发的 Activity 泄漏和 Bitmap 重复创建检测工具
- Trace Canary: 监控界面流畅性、启动耗时、页面切换耗时、慢函数及卡顿等问题
- SQLite Lint: 按官方最佳实践自动化检测 SQLite 语句的使用质量
- IO Canary: 检测文件 IO 问题，包括：文件 IO 监控和 Closeable Leak 监控



### javassit

javassit是一个开源的字节码创建、编辑类库，现属于Jboss web容器的一个子模块，特点是简单、快速，与AspectJ一样，使用它不需要了解字节码和虚拟机指令，这里是[官方文档](https://www.javassist.org/tutorial/tutorial.html)。

javassit核心的类库包含ClassPool，CtClass ，CtMethod和CtField。

- ClassPool：一个基于HashMap实现的CtClass对象容器。
- CtClass：表示一个类，可从ClassPool中通过完整类名获取。
- CtMethods：表示类中的方法。
- CtFields ：表示类中的字段。

### 总结

| 技术框架     | 特点                                                         | 开发难度 | 优势                                                     | 不足                                       |
| ------------ | ------------------------------------------------------------ | -------- | -------------------------------------------------------- | ------------------------------------------ |
| APT          | 常用于通过注解减少模板代码，对类的创建于增强需要依赖其他框架。 | ★★       | 开发注解简化上层编码。                                   | 使用注解对原工程具有侵入性。               |
| AspectJ      | 提供完整的面向切面编程的注解。                               | ★★       | 真正意义的AOP，支持通配、继承结构的AOP，无需硬编码切面。 | 重复织入、不织入问题                       |
| ASM          | 面向字节码指令编程，功能强大。                               | ★★★      | 高效，ASM5开始支持java8。                                | 切面能力不足，部分场景需硬编码。           |
| Javassit     | API简洁易懂，快速开发。                                      | ★        | 上手快，新人友好，具备运行时加载class能力。              | 切点代码编写需注意class path加载问题。     |
| java动态代理 | 运行时扩展代理接口功能。                                     | ★        | 运行时动态增强。                                         | 仅支持代理接口，扩展性差，使用反射性能差。 |

