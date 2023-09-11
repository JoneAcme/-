[TOC]





## Activity

### 生命周期

![](https://developer.android.google.cn/guide/components/images/activity_lifecycle.png?hl=zh-cn)

现有A、B两个Activity：

- 启动A：onCreate(A) -> onStart(A) -> onResume(A)
- 退到桌面：onPause(A) -> onStop(A)
- 从桌面返回到A：onRestart(A) -> onStart(A) -> onResume(A)
- A启动B：onPause(A) -> onCreate(B) -> onStart(B) -> onResume(B) -> onStop(A)
- B返回A：onPause(B) ->onRestart(A) -> onStart(A) -> onResume(A) -> onStop(B) -> onDestry(B)

**异常情况**

**onSaveInstanceState(Bundle outState)调用时机：**

1. **targetSdkVersion低于11的app，onSaveInstanceState方法会在Activity.onPause之前回调；**
2. **targetSdkVersion低于28的app，则会在onStop之前回调；**
3. **28之后，onSaveInstanceState在onStop回调之后才回调。**

1. A后台被杀死，重建：onCreate(A) –>onStart -> onRestoreInstanceState(A)

2. A横竖屏切换：

   - AndroidManifest没有设置configChanges：

      onPause ->onSaveInstanceState -> onStop -> onDestroy -> onCreate -> onStart -> onRestoreInstanceState -> onResume

   - configChanges android:configChanges="orientation"

      onPause ->onSaveInstanceState -> onStop -> onDestroy -> onCreate -> onStart -> onRestoreInstanceState -> onResume

   - configChanges android:configChanges="orientation|keyboardHidden|screenSize"

     只会调用：onConfigurationChanged

   - configChanges android:configChanges="orientation|screenSize"

     只会调用：onConfigurationChanged

   避免重建的方式：

   1. 固定屏幕方向： android:screenOrientation="portrait"
   2. 设置 android:configChanges="orientation|screenSize"

### 启动模式

#### 模式说明

| 模式           | 说明                                                         |
| -------------- | ------------------------------------------------------------ |
| standard       | 默认默认，每次启动Activity都会新建一个实例                   |
| singleTop      | 栈顶服用，启动时判断栈顶是否存在实例，如果存在，则复用该实例。生命周期不再调用onCreate、onStart，而是会调用onNewIntent |
| singleTask     | 栈内复用，如果当前栈内存在该Activity的实例，那么会移除该实例上的所有（默认具有clearTop效果）；如果栈内没有，则新建实例，压入对应栈 |
| singleInstance | 唯一实例，singleTask加强版，效果为singleTop+独立的栈         |

**注意**：
1. 每一个Activity启动时，都需要指定对应栈，默认为包名。
2. 在standard模式下，Activity会默认进入启动它的Activity所属的任务栈中。
3. 为什么使用Application、Service的Context启动Activity时，如果不指定FLAG_ACTIVITY_NEW_TASK，会ERROR，因为当前Context非Activity类型的Context，没有对应的任务栈。

#### 使用方法

| 方法                  |                                                 |
| --------------------- | ----------------------------------------------- |
| `AndroidMenifest.xml` | 无法指定clearTop选项（FLAG_ACTIVITY_CLEAR_TOP） |
| `Intent.addFlags()`   | 无法指定为SingleInstance 启动模式               |

其中，**Intent.addFlags 优先级更高**。

#### 重点参数说明

| 参数                 |                                                              |
| -------------------- | ------------------------------------------------------------ |
| TaskAffinity         | 为Activity的启动指定对应栈，默认为包名                       |
| AllowTaskReparenting | 跳转其他应用时，是否转移栈。 如：如 A应用跳转B应用的C界面，此时回到桌面，打开B应用，如果AllowTaskReparenting的值为true，那么B应用显示的为C界面而不是B应用的主页 |

#### 举例

**假设1**
有两个栈：前台任务栈： A -- B ，后台任务栈：C -- D ，上述描述右侧为栈顶，C、D的启动模式为SingTask，现B启动D，现在栈中是什么情况？
答：A -- B -- C -- D。前台任务栈中的B启动后台任务栈的D时，整个后台任务栈都会被切换到前台栈中

**假设2**
现有Activity ： A、B、C，包名为com.example.tast 
A ：launchMode = standard taskAffinity未指定
B、C ： launchMode = singleTask  taskAffinity = com.example.tast1
现 A-->B-->C-->A-->B

1. 当前栈中的实例分布

   |       | com.example.tast | com.example.tast1 |
   | ----- | ---------------- | ----------------- |
   | A     | A                |                   |
   | A-->B | A                | B                 |
   | B-->C | A                | B-->C             |
   | C-->A | A                | B-->C-->A         |
   | A-->B | A                | B                 |

   说明： 

   1. A未指定启动的栈，所以它的启动栈为启动它的Activity所在栈。
   2. 最后一步，B为SingleTask启动模式，默认具有clearTop效果，会将com.example.tast1栈中上方的C、A移除，故栈中只留B的实例

2. 按两次返回是什么效果

   第一次，com.example.tast1栈中B出栈，当前栈为空，com.example.tast 栈居顶部

   第二次，com.example.tast栈中A出栈，即显示桌面



### 启动其他应用的Activity？

在保证有权限访问的情况下，通过隐式Intent进行目标Activity的IntentFilter匹配，原则是：

- 一个intent只有同时匹配某个Activity的intent-filter中的action、category、data才算完全匹配，才能启动该Activity；
- 一个Activity可以有多个 intent-filter，一个 intent只要成功匹配任意一组 intent-filter，就可以启动该Activity；



------

## Service

### 生命周期

![](https://camo.githubusercontent.com/00a0bb98ef0d7d0f0539d5fc4d99e7c530401cf1/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f3934343336352d636635633161396432646464616163612e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f3435362f666f726d61742f77656270)

### 启动方式

| 启动方式     | 结束方式      | 对应生命周期                                                 |
| ------------ | ------------- | ------------------------------------------------------------ |
| startService | stopService   | startService -> onCreate -> onStartCommand -> StopService -> onDestroy |
| bindService  | unbindService | bindService -> onCreate -> onStartCommand -> onUnbind -> onDestroy |

### 注意条件

1. 每个Service只会存在一个实例。在整个生命周期内，只有`startCommand()`能被多次调用。其他方法只能被调用一次。（即只能绑定和解绑一次。）
2. 绑定后没有解绑，无法使用`stopService()`将其停止。
3. 如果已经`onCreate()`，那么`startService()`将只调用`startCommand()`。
4. 如果是以`bindService`开启，那么使用`unbindService`时就会自动调用`onDestroy`销毁。



------

## BroadcastReceiver

- Android8.0后，所有隐式广播都不允许使用静态注册的方式来接收了。

```java
LocalBroadcastManager.getInstance(MainActivity.this).registerReceiver(receiver, filter);
```



------

## ContentProvider

ContentProvider 管理对结构化数据集的访问。它们封装数据，并提供用于定义数据安全性的机制。 内容提供程序是连接一个进程中的数据与另一个进程中运行的代码的标准界面。

**ContentProvider 的 onCreate 要先于 Application 的 onCreate 而执行。**



------

## Fragment

![](https://camo.githubusercontent.com/d80aae526660b859bb231a9fc0bcc50fe55f8c7e/68747470733a2f2f646576656c6f7065722e616e64726f69642e676f6f676c652e636e2f696d616765732f667261676d656e745f6c6966656379636c652e706e67)

#### setArguments

**setArguments方法中传递的参数，可以保存，不会丢失**。fragment 的状态会保存在FragmentState（implements Parcelable）中，在重建时候由FragmentManager从FragmentState取出数据重新设置回去

> 当一个fragment重新创建的时候，系统会再次调用 Fragment中的**默认构造函数**。 
> 如果创建了一个带有重要参数的Fragment的之后，一旦由于什么原因（横竖屏切换）导致Fragment重新创建。
> 此时，重建默认调用无参的构造函数，也就是通过构造方法传递的参数会丢失。

#### FragmentTransaction管理的Fragment生命周期状态

##### add() VS replace()

只有在Fragment数量大于等于2的时候，调用add()还是replace()的区别才能体现出来。

| 连续两次添加Fragment | 区别                                                         |
| -------------------- | ------------------------------------------------------------ |
| add()                | 每个Fragment生命周期中的onAttach()-onResume()都会被各调用一次，而且两个Fragment的View会被同时attach到containerView中。退出Activty时，每个Fragment生命周期中的onPause() - onDetach()也会被各调用一次。 |
| replace()            | 第二次添加的Fragment会导致第一个Fragment被销毁（在不使用回退栈的情况下），即执行第二个Fragment的onAttach()方法之前会先执行第一个Fragment的onPause()-onDetach()方法，与此同时containerView会detach掉第一个Fragment的View。 |

##### show() & hide() VS attach() & detach()

| 调用                |                                                              |
| ------------------- | ------------------------------------------------------------ |
| show() & hide()     | **Fragment的正常生命周期方法并不会被执行**，仅仅是Fragment的View被显示或者隐藏，并视情况调用onHiddenChanged()。而且，尽管Fragment的View被隐藏，但它在父布局中并未被detach，仍然是作为containerView的childView存在着。 |
| attach() & detach() | 一旦一个Fragment被detach()，它的onPause()-onDestroyView()周期都会被执行，同时Fragment的View也会被detach，但是**不会执行onDestroy()和onDetach()**，也就是说Fragment的实例还是在内存中的。 |
| remove()            | 相对应add()方法执行onAttach()-onResume()的生命周期，remove()就是完成剩下的onPause()-onDetach()周期。 |
| replace()           | 其实就是remove()+add()。                                     |

#### FragmentPagerAdapter && FragmentStatePagerAdapter

|                           |                                                              |
| ------------------------- | ------------------------------------------------------------ |
| FragmentPagerAdapter      | 实际上是使用add()，attach()和detach()来管理Fragment的。缓冲范围内的从onAttach() - onResume()，超出缓存范围onPause() - onDestroyView()。 |
| FragmentStatePagerAdapter | 使用add()和remove()管理Fragment，所以缓存外的Fragment的实例不会保存在内存中 |



------

## View

### View绘制

ViewRoot 对应于 ViewRootImpl 类，它是连接 WindowManager 和 DecorView 的纽带，View 的三大流程均是通过 ViewRoot 来完成的。在 ActivityThread 中，当 Activity 对象被创建完毕后，会将 DecorView 添加到 Window 中，同时会创建 ViewRootImpl 对象，并将 ViewRootImpl 对象和 DecorView 建立关联

View 的整个绘制流程可以分为以下三个阶段：

| 阶段    | 说明                                             |
| ------- | ------------------------------------------------ |
| measure | 判断是否需要重新计算 View 的大小，需要的话则计算 |
| layout  | 判断是否需要重新计算 View 的位置，需要的话则计算 |
| draw    | 判断是否需要重新绘制 View，需要的话则重绘制      |

#### MeasureSpec
MeasureSpec 是 View 类的一个静态内部类，用来说明应该如何测量这个 View
MeasureSpec表示的是一个32位的整型值：

- 高2位表示测量模式SpecMode
- 低30位表示某种测量模式下的规格大小SpecSize

| Mode        | 说明                                                         |
| ----------- | ------------------------------------------------------------ |
| UNSPECIFIED | 不指定测量模式, 父视图没有限制子视图的大小，子视图可以是想要的任何尺寸，通常用于系统内部，应用开发中很少用到。 |
| EXACTLY     | 精确测量模式，视图宽高指定为 match_parent 或具体数值时生效，表示父视图已经决定了子视图的精确大小，这种模式下 View 的测量值就是 SpecSize 的值 |
| AT_MOST     | 最大值测量模式，当视图的宽高指定为 wrap_content 时生效，此时子视图的尺寸可以是不超过父视图允许的最大尺寸的任何尺寸 |

### 滑动速度：VelocityTracker

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

### 手势：GestureDetector

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