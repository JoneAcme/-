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



### 启动其他应用的Activity

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

### 生命周期

![](https://camo.githubusercontent.com/d80aae526660b859bb231a9fc0bcc50fe55f8c7e/68747470733a2f2f646576656c6f7065722e616e64726f69642e676f6f676c652e636e2f696d616765732f667261676d656e745f6c6966656379636c652e706e67)

### setArguments

**setArguments方法中传递的参数，可以保存，不会丢失**。fragment 的状态会保存在FragmentState（implements Parcelable）中，在重建时候由FragmentManager从FragmentState取出数据重新设置回去

> 当一个fragment重新创建的时候，系统会再次调用 Fragment中的**默认构造函数**。 
> 如果创建了一个带有重要参数的Fragment的之后，一旦由于什么原因（横竖屏切换）导致Fragment重新创建。
> 此时，重建默认调用无参的构造函数，也就是通过构造方法传递的参数会丢失。

### FragmentTransaction管理的生命周期

1. **add() VS replace()**

只有在Fragment数量大于等于2的时候，调用add()还是replace()的区别才能体现出来。

| 连续两次添加Fragment | 区别                                                         |
| -------------------- | ------------------------------------------------------------ |
| add()                | 每个Fragment生命周期中的onAttach()-onResume()都会被各调用一次，而且两个Fragment的View会被同时attach到containerView中。退出Activty时，每个Fragment生命周期中的onPause() - onDetach()也会被各调用一次。 |
| replace()            | 第二次添加的Fragment会导致第一个Fragment被销毁（在不使用回退栈的情况下），即执行第二个Fragment的onAttach()方法之前会先执行第一个Fragment的onPause()-onDetach()方法，与此同时containerView会detach掉第一个Fragment的View。 |

2. **show() & hide() VS attach() & detach()**

| 调用                |                                                              |
| ------------------- | ------------------------------------------------------------ |
| show() & hide()     | **Fragment的正常生命周期方法并不会被执行**，仅仅是Fragment的View被显示或者隐藏，并视情况调用onHiddenChanged()。而且，尽管Fragment的View被隐藏，但它在父布局中并未被detach，仍然是作为containerView的childView存在着。 |
| attach() & detach() | 一旦一个Fragment被detach()，它的onPause()-onDestroyView()周期都会被执行，同时Fragment的View也会被detach，但是**不会执行onDestroy()和onDetach()**，也就是说Fragment的实例还是在内存中的。 |
| remove()            | 相对应add()方法执行onAttach()-onResume()的生命周期，remove()就是完成剩下的onPause()-onDetach()周期。 |
| replace()           | 其实就是remove()+add()。                                     |

### Adapter对比

|                           |                                                              |
| ------------------------- | ------------------------------------------------------------ |
| FragmentPagerAdapter      | 实际上是使用add()，attach()和detach()来管理Fragment的。缓冲范围内的从onAttach() - onResume()，超出缓存范围onPause() - onDestroyView()。 |
| FragmentStatePagerAdapter | 使用add()和remove()管理Fragment，所以缓存外的Fragment的实例不会保存在内存中 |



------

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



------

## View

### View绘制

ViewRoot 对应于 ViewRootImpl 类，它是连接 WindowManager 和 DecorView 的纽带，View 的三大流程均是通过 ViewRoot 来完成的。在 ActivityThread 中，当 Activity 对象被创建完毕后，会将 DecorView 添加到 Window 中，同时会创建 ViewRootImpl 对象，并将 ViewRootImpl 对象和 DecorView 建立关联。绘制的入口是由**ViewRootImpl**的performTraversals方法来发起

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
| EXACTLY     | 精确测量模式，视图宽高指定为 match_parent 或具体数值时生效，表示父视图已经决定了子视图的精确大小，这种模式下 View 的测量值就是 SpecSize 的值（LayoutParams中的宽高） |
| AT_MOST     | 最大值测量模式，当视图的宽高指定为 wrap_content 时生效，此时子视图的尺寸可以是不超过父视图允许的最大尺寸的任何尺寸 |

#### Darw顺序

1. drawBackground ： 绘制背景
2. 如有必要，颜色渐变淡之前保存画布层(即锁定原有的画布内容)
3. onDraw ： 绘制内容
4. dispatchDraw：绘制子View（如果是ViewGroup，才会有此方法）
5. 如有必要，绘制颜色渐变淡的边框，并恢复画布(即画布改变的内容附加到原有内容上)
6. Draw decorations (scrollbars for instance)   绘制装饰，比如滚动条

#### 注意事项

关于绘制方法，有两点需要注意一下：

1. 出于效率、简化绘制流程，ViewGroup 默认会绕过 draw() 方法，换而直接执行 dispatchDraw()。如果需要在ViewGroup中绘制内容，需要调用 **View.setWillNotDraw(false)** 这行代码来切换到完整的绘制流程（有些 ViewGroup 是已经调用过 setWillNotDraw(false) 了的，例如 ScrollView）。
2. 绘制代码优先写在 onDraw() ，onDraw方法中可以在不需要重绘的时候自动跳过重复执行，以提升开发效率。

### View 的事件分发

#### 发起

1. 事件最先传到Activity的dispatchTouchEvent()进行事件分发
2. 调用Window类实现类PhoneWindow的superDispatchTouchEvent()
3. 调用DecorView的superDispatchTouchEvent()
4. 最终调用DecorView父类的dispatchTouchEvent()，即ViewGroup的dispatchTouchEvent()

#### 传递

主要包含3个方法：

| 方法                  | 说明                                   |
| --------------------- | -------------------------------------- |
| dispatchTouchEvent    | 分发触摸事件                           |
| onInterceptTouchEvent | 判断是否拦截改事件（ViewGroup）        |
| onTouchEvent          | 处理点击事件，返回值代表是否消费该事件 |

首先是调用顶层ViewGroup 的dispatchTouchEvent 方法开始分发。

dispatchTouchEvent由上至下调用，onTouchEvent由下至上调用。

onInterceptTouchEvent返回值：

- true：拦截事件并交由本身的onTouchEvent处理，

- false：不拦截，遍历子View的onTouchEvent

**常用处理冲突方法：**

requestDisallowInterceptTouchEvent：

1.在down时调用getParent().requestDisallowInterceptTouchEvent(true),取消父类对事件的拦截

2.在up、cancel时调用getParent().requestDisallowInterceptTouchEvent(false)

每次Down时，拦截状态会被重置，所以要在Down事件中重新赋值。

#### 常见问题

1. **CANCEL事件时机**：

   - 事件被父控件拦截，比如控件接收到了Down事件，但是父控件拦截了Up事件，此时Up事件就会被转发为Cancel事件分发给子View
   - 事件流程被异常情况打断，比如处理事件的过程中，系统更新功能弹出，意外发生了界面切换，再回到APP时，会补发一个Cancel事件来取消之前的操作
   -  事件处理过程中，被父控件移除或被从Window中移除
   - 如果当前点击的View是空的（空白处），或者说当前的事件没有View接收

   要特别注意的：

   - 滑出view范围后，如果父view没有拦截事件，则会继续受到`ACTION_MOVE`和`ACTION_UP`等事件。

   - 一旦滑出view范围，view会被移除`PRESSED`标记，这个是不可逆的，然后在`ACTION_UP`中不会执行`performClick()`等逻辑。

### LayoutInflater 过程

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



------

## RecyclerView

RecyclerView的缓存分为四级，缓存复用机制针对复用的内容是**ViewHolder**

- **Scrap**

  屏幕内的缓存数据，只在布局、预布局时使用，不参与滚动时的回收复用，这些ViewHolder是无效、未移除、未标记的

  mAttachedScrap：没有改变的ViewHolder

  mChangedScrap：发生变化的ViewHolder

- **CacheView**

  **最近**移出屏幕的缓存数据，**默认大小是2个**。

  根据FIFO原则，把优先进入的缓存数据移出并放到下一级缓存中，然后再把新的数据添加进来。

  数据包含了原来的ViewHolder的所有数据信息，数据可以直接来拿来复用。

  **根据position来寻找数据**（根据用户是上划/下滑），取出的ViewHolder可直接使用。 如：顶部可见position=1， 下拉时，优先获取position = 0的ViewHolder。

- **ViewCacheExtension**

  需要自定义，不明了，慎用

  

- **RecycledViewPool**

  Cache满了后，存入RecycledViewPool，**默认大小是5个**。

  数据会被重置**。**RecycledViewPool是根据itemType获取的

  根据itemType获取，取出的ViewHolder需要重新进行数据绑定（`onBindViewHolder()`）

  

**预布局(PreLayout)**

- 真正布局之前，事先布局一次
- 预布局包含被移除的Item
- 预布局与真正布局对比，用来做动画



------

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



**webp**

基于VP8视频编码中的预测编码方法来压缩图像数据，支持无损压缩和透明色的功能。 WebP 无损对 PNG 图片的优化到达了 20%~40%。



------

## 动画

|          |                                                              | 场景                                                         | 针对视图而言                                                 |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 补间动画 | 平移、缩放、旋转、透明度。<br />作用于视图View，不可用于属性如颜色、长度等 | `Activity` 的切换效果、<br />`Fragement` 的切换效果、<br />视图组（`ViewGroup`）中子元素的出场效果 | 无改变视图的属性，<br />仅对视图进行变换，<br />响应区域还在原位 |
| 帧动画   | 图片动画，容易oom，<br />作用于视图View，不可用于属性如颜色、长度等 | 较为复杂的个性化动画效果                                     | 无改变视图的属性，<br />仅对视图进行变换，<br />响应区域还在原位 |
| 属性动画 | 任意Java对象，不局限于视图控件                               | 不断对值进行改变并赋值给对象的属性，可以是任意对象的任意属性（需要有set方法） | 改变了视图属性<br />响应区域随动画更新                       |

**属性动画的一些使用方式**

![img](https://imgconvert.csdnimg.cn/aHR0cDovL3VwbG9hZC1pbWFnZXMuamlhbnNodS5pby91cGxvYWRfaW1hZ2VzLzk0NDM2NS1lNzVlMzI4ZjdlM2ZhYjU5LnBuZz9pbWFnZU1vZ3IyL2F1dG8tb3JpZW50L3N0cmlwJTdDaW1hZ2VWaWV3Mi8yL3cvMTI0MA)





------

## Handler

### 机制

- Message：Handler 接收和处理的消息对象
- MessageQueue：Message 的队列，先进先出，每一个线程最多可以拥有一个，通过**单链表**的数据结构来维护消息列表，因为单链表在插入和删除上比较有优势。
- Looper：消息泵，是 MessageQueue 的管理者，会不断从 MessageQueue 中取出消息，并将消息分给对应的 Handler 处理，**每个线程只有一个 Looper**，**保存在ThreadLocal中**。**主线程：ActivityThread 被创建的时候就会初始化 Looper，可直接使用如果在子线程中使用，需要手动调用Looper.prepare() 、Looper.loop()！**

总结： 一个线程对应一个 Looper 对应一个 MessageQueue 对应多个 Handler。

每个线程中可以有多个Handler，即一个Looper可以处理来自多个Handler的消息。 Looper中维护一个MessageQueue，来维护消息队列，消息队列中的Message可以来自不同的Handler。

![](https://pic4.zhimg.com/80/v2-e75d7338f3ad8c8afe3d83b41adf695b_1440w.webp)

调用时序：

1. `MessageQueue.enqueueMessage`，向消息队列中添加消息
2. 通过 `Looper.loop`开启循环后，会不断地从消息池中读取消息，即调用 `MessageQueue.next`
3. 调用目标Handler的 `dispatchMessage`方法传递消息
4. 返回到Handler所在线程，调用 `handleMessage`方法，处理消息

**延迟消息**

1. 将传入的延迟时间转化成距离开机时间的毫秒数
2. MessageQueue 中根据上一步转化的时间进行顺序排序
3. 在 MessageQueue.next 获取消息时，对比当前时间（now）和第一步转化的时间（when），如果 now < when，则通过 epoll_wait 的 timeout 进行等待
4. 如果该消息需要等待，会进行 idel handlers 的执行，执行完以后会再去检查此消息是否可以执行

### 关于Looper死循环

looper中loop 方法死循环调用MessageQueue.next() 获取消息。

1. **Linux epoll机制**

   当主线程的MessageQueue没有消息时，便阻塞在loop的queue.next()中的nativePollOnce()方法里（native方法，使用**Linux epoll机制**： 是一种IO多路复用机制，可以同时监控多个描述符，当某个描述符就绪(读或写就绪)，则立刻通知相应程序进行读或写操作，本质同步I/O，即读写是阻塞的）

   因此，此时主线程会释放CPU资源进入休眠状态，直到下个消息到达或者有事务发生，通过往pipe管道写端写入数据来唤醒主线程工作。所以，主线程大多数时候都是处于休眠状态，并不会消耗大量CPU资源。

2. **为何不会阻塞主线程**

   Android 的是由事件驱动的，looper.loop() 不断地接收事件、处理事件，每一个点击触摸或者说Activity的生命周期都是运行在 Looper.loop() 的控制之下，如果它停止了，应用也就停止了。只能是某一个消息或者说对消息的处理阻塞了 Looper.loop()，而不是 Looper.loop() 阻塞它。

3. **为什么不用 java 中的 wait / notify 而用 epoll 呢**

    Android 2.2 及以前使用的是： java 中的 wait / notify 也能实现阻塞等待消息的功能

   因为要处理native 侧的事件，故换为epoll

4. [**为什么不用 wait 而用 epoll 呢？**](https://mp.weixin.qq.com/s/2-MBLmMqUr_GARVEi81Ntg)

    java 中的 wait / notify 也能实现阻塞等待消息的功能，在 Android 2.2 及以前，也确实是这样做的。

   需要处理 native 侧的事件，所以只使用 java 的 wait / notify 就不够用了。

   不过这里最开始使用的还是 select（遍历），后面才改成 epoll。

   - select 无法定位是哪个socket唤醒的，需要遍历两次定位socket
   - epoll 可高效地监视多个 socket。内核维护一个“就绪列表”，引用收到数据的 socket，就能避免遍历。

   [select、poll和epoll区别](https://zhuanlan.zhihu.com/p/272891398)

### 同步屏障

[参考链接](https://www.cnblogs.com/renhui/p/12875589.html)

Handler的Message种类分为3种：

- 普通消息（同步消息）
- 屏障消息（同步屏障）
- 异步消息

屏障消息：插入一个屏障，阻拦之后普通消息，只能处理异步消息。即**屏障消息就是为了确保异步消息的优先级，设置了屏障后，只能处理其后的异步消息，同步消息会被挡住，除非撤销屏障。**

屏障消息：不需要被分发，没有tartget

普通消息：需要分发，需要target

**细节说明**：

- postSyncBarrier方法就是用来插入一个屏障到消息队列的
- 屏障和普通消息一样可以根据时间来插入到消息队列中的适当位置，并且只会挡住它后面的同步消息的分发。
- postSyncBarrier返回一个int类型的数值，通过这个数值可以撤销屏障。
- postSyncBarrier方法是私有的，如果我们想调用它就得使用反射。
- 插入普通消息会唤醒消息队列，但是插入屏障不会。

原理：

1. MessageQueue.next 中判断是否屏障消息（根据target是否为空）

2. 当前为屏障消息，遍历消息链表找到最近的一条异步消息。

   存在异步消息：

   case1：到了处理时间，返回该消息；

   case2：未到处理时间，继续等待

3. 非屏障消息，按顺序获取下一条消息。

### 常见问题

1. 多个线程给 MessageQueue 发消息，如何保证线程安全？

   MessageQueue 使用 synchronized (this) 加锁

2. View.post 和 Handler.post 的区别

   View.post 都是通过 ViewRootImpl 内部的 Handler 进行处理的。

   1. 如果在 performTraversals 前调用 View.post，则会将消息进行保存，之后在 dispatchAttachedToWindow 的时候通过 ViewRootImpl 中的 Handler 进行调用。
   2. 如果在 performTraversals 以后调用 View.post，则直接通过 ViewRootImpl 中的 Handler 进行调用。

3. View.post 里可以拿到 View 的宽高信息

   View.post 的 Runnable 执行的时候，已经执行过 performTraversals 了，也就是 View 的 measure layout draw 方法都执行过了

4. 非 UI 线程真的不能操作 View 吗

   检查，其实并不是检查主线程，是检查 mThread != Thread.currentThread，而 mThread 指的是 ViewRootImpl 创建的线程。

   所以非 UI 线程确实不能操作 View，但是检查的是创建的线程是否是当前线程，因为 ViewRootImpl 创建是在主线程创建的，所以在非主线程操作 UI 过不了这里的检查。

### 内存泄漏

**场景**：

- 延迟消息：handler作为Activity内部类
- 延迟消息：handler dispatchMessage中持有Activity引用

**原因**：延迟消息执行前，handler持有Activity引用，此时Activity Finish，没办法回收

**解决**：

1. Handler的具体实现使用静态内部类的方式（静态内部类不会持有外部类的引用）

2. 再对外部的Activity进行弱引用。

3. 若是Handler有延时操作，那么Handler依然会存在，如果延时消息在Activity销毁之后没有存在的必要，那么我们可以在Activity销毁之后删除Handler的消息。

   

------

## HandlerThread

提供了Looper的Thread。

**内部原理 = `Thread`类 + `Handler`类机制**，即：

- 通过继承`Thread`类，**快速地创建1个带有Looper对象的新工作线程**
- 通过封装`Handler`类，**快速创建Handler& 与其他线程进行通信**

**优点**：只要开启一个线程，就可以处理多个耗时任务。

**缺点**：任务是串行执行的，不能并行执行。一旦队列中有某个任务执行时间过长，就会导致后续的任务都会被延迟处理。

**注意事项**

1. HandlerThread不再需要使用的时候，要调用`quitSafe()`或者`quit()`方法来结束线程。
2. `quitSafe()`会等待正在处理的消息处理完后再退出，而`quit()`不管是否正在处理消息，直接移除所有回调。



------

## IntentService

- **作用**：Service 自动实现多线程的子类，处理异步请求 & 实现多线程

- **使用**：

  1. 定义IntentService的子类，传入线程的名称，重写onHandleIntent()方法。
  2. AndroidManifest.xml注册Service。
  3. 开启Service。

- **本质**：`Service`+ `Handler`+ `HandlerThread`

- **优点**：

  本身是Service，优先级高于线程

  执行完成后自动结束

  适合 **按顺序**、**在后台执行**的耗时任务，如：离线下载

- **机制**：

  1. 新建HandlerThread

  2. 持有内部内部类（Handler），其Looper绑定HandlerThread的工作线程

  3. 多次启动IntentService，都会调用其onStart（调用自封装的startCommand()方法）

  4. onStart中通过内部类（Handler）发送、切换到对应工作线程

  5. 在工作线程中执行onHandleIntent()

  6. 执行完onHandleIntent方法后，会调用stopSelf结束本服务，无需手动结束服务

     

  |               | 关系          | 运行线程 | 结束服务        |
  | ------------- | ------------- | -------- | --------------- |
  | Service       |               | 主线程   | 手动stopService |
  | IntentService | 继承自Service | 工作线程 | 执行完自动停止  |

  - IntentService 实现Service中的onBind，默认返回null
  - IntentService 实现Service中的onStartCommand，将请求的Intent放入消息队列中等待执行

  **注意事项**

  1. 不建议通过 bindService() 启动 IntentService
     - IntentService 实现Service中的onBind，默认返回null
     - 采用 `bindService()`启动 `IntentService`的生命周期：onCreate() ->> onBind() ->> onunbind()->> onDestory()；不会调用onStartCommand，无法将请求的Intent放入消息队列中

  

------

## Android中的多进程

1. **指定四大组件的android:process属性**

   android:process属性取值：

   - xxx.xxx.xxx.new

   - :new

     “:”的含义：

     1. 指要在当前的进程名前面附加上包名
     2. 以“:”开头的进程属于当前应用的私有线程，其他应用不能与它跑在同一进程中
     3. 而不以“:”开头的进程属于全局线程，其他应用通过ShareUID的方式可以和它跑在同一进程中

2. **通过JNI在native层fork一个新的进程**

多进程会造成的问题：

- 静态成员和单例模式完全失效
- 线程同步机制完全失效
- SharedPerences可靠性下降
- Application会多次创建



------

### 序列化

Serializable：Java序列化接口，写入文件保存、传输

Parcelable：Android序列号接口，写入内存直接传输

|              | 介绍                                | 存储位置                  | 是否需要反射                                    |
| ------------ | ----------------------------------- | ------------------------- | ----------------------------------------------- |
| Serializable | Java序列化接口，写入文件保存、传输  | 使用 I/O 读写存储在硬盘上 | 使用反射，序列化和反序列化过程需要大量 I/O 操作 |
| Parcelable   | Android序列号接口，写入内存直接传输 | 直接在内存中读写          | 不需要用反射，数据也存放在 Native 内存中        |

### IPC方式

| 名称            | 优点                                                         | 缺点                                                         | 适用场景                                                     |
| --------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Bundle          | 简单易用                                                     | 只能传输 Bundle 支持的数据类型                               | 四大组件间的进程间通信                                       |
| 文件共享        | 简单易用                                                     | 不适合高并发场景，并且无法做到进程间即时通信                 | 无并发访问情形，交换简单的数据实时性不高的场景               |
| AIDL            | 功能强大，支持一对多并发通信，支持实时通信                   | 使用稍复杂，需要处理好线程同步                               | 一对多通信且有 RPC 需求                                      |
| Messenger       | 功能一般，支持一对多串行通信，支持实时通信                   | 不能很处理高并发清醒，不支持 RPC，数据通过 Message 进行传输，因此只能传输 Bundle 支持的数据类型 | 低并发的一对多即时通信，无RPC需求，或者无需返回结果的RPC需求 |
| ContentProvider | 在数据源访问方面功能强大，支持一对多并发数据共享，可通过 Call 方法扩展其他操作 | 可以理解为受约束的 AIDL，主要提供数据源的 CRUD 操作          | 一对多的进程间数据共享                                       |
| Socket          | 可以通过网络传输字节流，支持一对多并发实时通信               | 实现细节稍微有点烦琐，不支持直接的RPC                        | 网络数据交换                                                 |

### Linux内存映射

[参考链接](https://www.jianshu.com/p/719fc4758813)

![内存映射示意图](https://upload-images.jianshu.io/upload_images/944365-d5a20d7c6c16ead5.png?imageMogr2/auto-orient/strip|imageView2/2/w/510/format/webp)

**实现过程**：

- 内存映射的实现过程主要是通过`Linux`系统下的系统调用函数：`mmap（）`
- 该函数的作用 = 创建虚拟内存区域 + 与共享对象建立映射关系

**特点**：

- 提高数据的读、写 & 传输的时间性能
  1. 减少了数据拷贝次数
  2. 用户空间 & 内核空间的高效交互（通过映射的区域 直接交互）
  3. 用内存读写 代替 I/O读写
- 提高内存利用率：通过虚拟内存 & 共享对象

#### 文件操作

1. 传统的`Linux`系统文件操作

![](https://upload-images.jianshu.io/upload_images/944365-c2605f7bb79b0865.png?imageMogr2/auto-orient/strip|imageView2/2/w/960/format/webp)

2. 内存映射文件操作

![](https://upload-images.jianshu.io/upload_images/944365-7f0c6c23bb3d1cb9.png?imageMogr2/auto-orient/strip|imageView2/2/w/940/format/webp)

#### 跨进程通信

1. 传统跨进程通信

   ![传统跨进程通信](https://upload-images.jianshu.io/upload_images/944365-d3d15895eb9a58e6.png?imageMogr2/auto-orient/strip|imageView2/2/w/1030/format/webp)

2. 内存映射跨进程通信

   ![内存映射跨进程通信](https://upload-images.jianshu.io/upload_images/944365-df2a3cb545cb59ea.png?imageMogr2/auto-orient/strip|imageView2/2/w/960/format/webp)

### 

###  Binder机制

Android中的实现主要依靠 Binder类（内存映射），其实现了IBinder 接口

流程：

1. 创建AIDL接口
2. 创建、注册远程服务
3. 绑定远程服务

Binder 优势：

- Binder 本身是 C/S 架构的，更符合 Android 系统的架构
- 性能上更有优势：管道，消息队列，Socket 的通讯都需要两次数据拷贝，而 Binder 只需要一次。要知道，对于系统底层的 IPC 形式，少一次数据拷贝，对整体性能的影响是非常之大的
- 安全性更好：传统 IPC 形式，无法得到对方的身份标识（UID/GID)，而在使用 Binder IPC 时，这些身份标示是跟随调用过程而自动传递的。Server 端很容易就可以知道 Client 端的身份，非常便于做安全检查

[一次Binder通信最大可以传输多大的数据](https://www.jianshu.com/p/ea4fc6aefaa8) ： 1MB-8KB（PS：8k是两个pagesize，一个pagesize是申请物理内存的最小单元），