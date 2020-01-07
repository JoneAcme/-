# Jetpack 之 Lifecycle



[TOC]

## 简介

       用于观察Activity、Fragment 的生命周期，避免复写生命周期方法造成的臃肿问题。例在MVP 的Presenter中监听，Presenter中对应界面生命周期执行一些操作

## 使用

1. 监听Activity、Fragment 生命周期的类（例如Presenter）实现`LifecycleObserver`接口。

2. 添加要监听的生命周期方法

   方法添加  `@OnLifecycleEvent` 注解

   ```kotlin
   interface IPresenter : LifecycleObserver {
       @OnLifecycleEvent(Lifecycle.Event.ON_CREATE)
       fun onLifeCreate() {
       }
   
       @OnLifecycleEvent(Lifecycle.Event.ON_RESUME)
       fun onLifeResume() {
       }
   
       @OnLifecycleEvent(Lifecycle.Event.ON_ANY)
       fun onLifeAny() {
   
       }
   }
   class BasePrenter : IPresenter {
   
   }
   ```

3. FragmentActivity、Fragment 中 

   ```
    private val mPresenter = BasePrenter()
    override fun onCreate(savedInstanceState: Bundle?) {
           super.onCreate(savedInstanceState)
           setContentView(R.layout.activity_main)
           // 添加监听类
           lifecycle.addObserver(mPresenter)
    }
   ```



可订阅的所有事件：

```java
public enum Event {
    /**
     * Constant for onCreate event of the {@link LifecycleOwner}.
     */
    ON_CREATE,
    /**
     * Constant for onStart event of the {@link LifecycleOwner}.
     */
    ON_START,
    /**
     * Constant for onResume event of the {@link LifecycleOwner}.
     */
    ON_RESUME,
    /**
     * Constant for onPause event of the {@link LifecycleOwner}.
     */
    ON_PAUSE,
    /**
     * Constant for onStop event of the {@link LifecycleOwner}.
     */
    ON_STOP,
    /**
     * Constant for onDestroy event of the {@link LifecycleOwner}.
     */
    ON_DESTROY,
    /**
     * An {@link Event Event} constant that can be used to match all events.
     */
    ON_ANY
}
```

   

lifecycle状态：

```java
public enum State {
    /**
     * Destroyed state for a LifecycleOwner. After this event, this Lifecycle will not dispatch
     * any more events. For instance, for an {@link android.app.Activity}, this state is reached
     * <b>right before</b> Activity's {@link android.app.Activity#onDestroy() onDestroy} call.
     */
    DESTROYED,

    /**
     * Initialized state for a LifecycleOwner. For an {@link android.app.Activity}, this is
     * the state when it is constructed but has not received
     * {@link android.app.Activity#onCreate(android.os.Bundle) onCreate} yet.
     */
    INITIALIZED,

    /**
     * Created state for a LifecycleOwner. For an {@link android.app.Activity}, this state
     * is reached in two cases:
     * <ul>
     *     <li>after {@link android.app.Activity#onCreate(android.os.Bundle) onCreate} call;
     *     <li><b>right before</b> {@link android.app.Activity#onStop() onStop} call.
     * </ul>
     */
    CREATED,

    /**
     * Started state for a LifecycleOwner. For an {@link android.app.Activity}, this state
     * is reached in two cases:
     * <ul>
     *     <li>after {@link android.app.Activity#onStart() onStart} call;
     *     <li><b>right before</b> {@link android.app.Activity#onPause() onPause} call.
     * </ul>
     */
    STARTED,

    /**
     * Resumed state for a LifecycleOwner. For an {@link android.app.Activity}, this state
     * is reached after {@link android.app.Activity#onResume() onResume} is called.
     */
    RESUMED;

    /**
     * Compares if this State is greater or equal to the given {@code state}.
     *
     * @param state State to compare with
     * @return true if this State is greater or equal to the given {@code state}
     */
    public boolean isAtLeast(@NonNull State state) {
        return compareTo(state) >= 0;
    }
}
```

## 源码解析

- **ReportFragmeng** 

  监听生命周期的具体操作类

- **LifecycleObserver接口（ Lifecycle观察者）**

  实现该接口的类，通过注解的方式，可以通过被**LifecycleOwner**类的addObserver(LifecycleObserver o)方法注册,被注册后，LifecycleObserver便可以观察到LifecycleOwner的**生命周期事件**。

- **LifecycleOwner接口（Lifecycle持有者）**

  实现该接口的类持有**生命周期**(Lifecycle对象)，该接口的生命周期(Lifecycle对象)的改变会被其注册的观察者LifecycleObserver观察到并触发其对应的事件。

- **Lifecycle(生命周期)**

  和LifecycleOwner不同的是，LifecycleOwner本身持有Lifecycle对象，LifecycleOwner通过其Lifecycle getLifecycle()的接口获取内部Lifecycle对象。

- **State(当前生命周期所处状态)**

- **Event(当前生命周期改变对应的事件)**

  



Lifecycle:

```java
public abstract class Lifecycle {

        //注册LifecycleObserver （比如Presenter）
        public abstract void addObserver(@NonNull LifecycleObserver observer);
        //移除LifecycleObserver
        public abstract void removeObserver(@NonNull LifecycleObserver observer);
        //获取当前状态
        public abstract State getCurrentState();

        public enum Event {
            ON_CREATE,
            ON_START,
            ON_RESUME,
            ON_PAUSE,
            ON_STOP,
            ON_DESTROY,
            ON_ANY
        }

       public enum State {
            DESTROYED,
            INITIALIZED,
            CREATED,
            STARTED,
            RESUMED;

            public boolean isAtLeast(@NonNull State state) {
                return compareTo(state) >= 0;
            }
       }
}
```



LifecycleOwner:

```java
public interface LifecycleOwner {
    /**
     * Returns the Lifecycle of the provider.
     *
     * @return The lifecycle of the provider.
     */
    @NonNull
    Lifecycle getLifecycle();
}
```





<font color="ff0f0f">可直接使用监听的`AppCompatActivity` 继承关系：</font>

   `Activity`

       `SupportActivity`
    
           `FragmentActivity`
    
               `AppCompatActivity`



<font color="ff0f0f"> **入口：SupportActivity中的源码：**</font>

```java
public class SupportActivity extends Activity implements LifecycleOwner, Component {
    ....
 protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        // 此处 注入一个ReportFragment
        ReportFragment.injectIfNeededIn(this);
    }
    ...
}
```



```java
// ReportFragment.java

public class ReportFragment extends Fragment {
        ......
    public static void injectIfNeededIn(Activity activity) {
        // ProcessLifecycleOwner should always correctly work and some activities may not extend
        // FragmentActivity from support lib, so we use framework fragments for activities
        android.app.FragmentManager manager = activity.getFragmentManager();
        if (manager.findFragmentByTag(REPORT_FRAGMENT_TAG) == null) {
            manager.beginTransaction().add(new ReportFragment(), REPORT_FRAGMENT_TAG).commit();
            // Hopefully, we are the first to make a transaction.
            manager.executePendingTransactions();
        }
    }
  ......
}
```

       <font color="fa0faf"> 其实就是在Activity中添加一个Fragment,该Fragment用来监听activity生命周期的变化并通知观察者</font>

1. Fragment的onCreate()、onStart()、onResume()方法均在其所在的Activity的相应的onCreate()、onStart()、onResume()方法之后执行
2. 但是Fragment的onPause()、onStop()、onDestroy()均在所在Activit相应的onPause()、onStop()、onDestroy()之前执行



<font color="ff0f0f"> **`ReportFragment` 生命周期都做了什么操作：**</font>

```java
@Override
public void onActivityCreated(Bundle savedInstanceState) {
    super.onActivityCreated(savedInstanceState);
    //1. 分发事件
    dispatchCreate(mProcessListener);
    dispatch(Lifecycle.Event.ON_CREATE);
}

@Override
public void onStart() {
    super.onStart();
    dispatchStart(mProcessListener);
    dispatch(Lifecycle.Event.ON_START);
}
//2. 接口分发
private void dispatchCreate(ActivityInitializationListener listener) {
    if (listener != null) {
        listener.onCreate();
    }
}
……
```

mProcessListener来看看是在哪里设置的:

```java
/**
 * Class that provides lifecycle for the whole application process.
 */
public class ProcessLifecycleOwner implements LifecycleOwner {
    
    //注意,我是一个单例
    private static final ProcessLifecycleOwner sInstance = new ProcessLifecycleOwner();

    static void init(Context context) {
        sInstance.attach(context);
    }

    void attach(Context context) {
        mHandler = new Handler();
        mRegistry.handleLifecycleEvent(Lifecycle.Event.ON_CREATE);
        Application app = (Application) context.getApplicationContext();
        app.registerActivityLifecycleCallbacks(new EmptyActivityLifecycleCallbacks() {
            @Override
            public void onActivityCreated(Activity activity, Bundle savedInstanceState) {
                
// 这里设置的
              ReportFragment.get(activity).setProcessListener(mInitializationListener);
            }
    
            @Override
            public void onActivityPaused(Activity activity) {
                activityPaused();
            }
    
            @Override
            public void onActivityStopped(Activity activity) {
                activityStopped();
            }
        });
    }
}

//这里创建的 Activity的监听器
ActivityInitializationListener mInitializationListener =
            new ActivityInitializationListener() {
                @Override
                public void onCreate() {
                }

                @Override
                public void onStart() {
                    activityStarted();
                }

                @Override
                public void onResume() {
                    activityResumed();
                }

private final LifecycleRegistry mRegistry = new LifecycleRegistry(this);

//Activity创建的时候,分发Lifecycle.Event.ON_START事件
void activityStarted() {
    mStartedCounter++;
    if (mStartedCounter == 1 && mStopSent) {
        mRegistry.handleLifecycleEvent(Lifecycle.Event.ON_START);
        mStopSent = false;
    }
}
```

   

```java
//ReportFragment.java
static ReportFragment get(Activity activity) {
    return (ReportFragment) activity.getFragmentManager().findFragmentByTag(
            REPORT_FRAGMENT_TAG);
}
```

       ProcessLifecycleOwner给整个APP提供lifecycle的,也就是说通过它可以观察到整个应用程序的生命周期. 
    
       ProcessLifecycleOwner的attach()中registerActivityLifecycleCallbacks()注册了一个监听器,**一旦有Activity创建就给它设置一个Listener**.这样就保证了每个ReportFragment都有Listener.



       既然是一个全局的单例,并且可以监听整个应用程序的生命周期,那么,肯定一开始就需要初始化. 既然没有让我们在Application里面初始化,那么肯定就是在ContentProvider里面初始化的.
    
       **ContentProvider的onCreate()方法执行时间比Application的onCreate()执行时间还要早,而且肯定会执行.所以在ContentProvider的onCreate()方法里面初始化**

```java
public class ProcessLifecycleOwnerInitializer extends ContentProvider {
    @Override
    public boolean onCreate() {
        LifecycleDispatcher.init(getContext());
        ProcessLifecycleOwner.init(getContext());
        return true;
    }
}
```

<font color="ff0f0f">       **ProcessLifecycleOwner初始化,是拿来观察整个应用的生命周期的,其原理就是利用ReportFragment**</font>

看一下`LifecycleDispatcher`是做什么的

```java
class LifecycleDispatcher {
    static void init(Context context) {
        ...
        //registerActivityLifecycleCallbacks  注册一个监听器
        ((Application) context.getApplicationContext())
                .registerActivityLifecycleCallbacks(new DispatcherActivityCallback());
    }
}
static class DispatcherActivityCallback extends EmptyActivityLifecycleCallbacks {
    @Override
    public void onActivityCreated(Activity activity, Bundle savedInstanceState) {
        //又注入一次
        ReportFragment.injectIfNeededIn(activity);
    }
    @Override
    public void onActivityStopped(Activity activity) {
    }
    @Override
    public void onActivitySaveInstanceState(Activity activity, Bundle outState) {
    }
}
```

       在ComponentActivity中，每个Activity都注册了，这里又注册一次，个人猜测，可能是为了避免注册不成功吧，多次注册injectIfNeededIn 也只会注入成功一次。



**回到`ReportFragment`生命周期中，事件分发：**

```java
//ReportFragment.java

 private void dispatch(Lifecycle.Event event) {
        Activity activity = getActivity();
        if (activity instanceof LifecycleRegistryOwner) {
            ((LifecycleRegistryOwner) activity).getLifecycle().handleLifecycleEvent(event);
            return;
        }

        if (activity instanceof LifecycleOwner) {
            Lifecycle lifecycle = ((LifecycleOwner) activity).getLifecycle();
            if (lifecycle instanceof LifecycleRegistry) {
                ((LifecycleRegistry) lifecycle).handleLifecycleEvent(event);
            }
        }
    }
   
```

`((LifecycleRegistry) lifecycle).handleLifecycleEvent(event);`分发事件

```java
// LifecycleRegistry.java
//分发事件
public void handleLifecycleEvent(@NonNull Lifecycle.Event event) {
        State next = getStateAfter(event);
        moveToState(next);
    }

private void moveToState(State next) {
        if (mState == next) {
            return;
        }
        mState = next;
        if (mHandlingEvent || mAddingObserverCounter != 0) {
            mNewEventOccurred = true;
            // we will figure out what to do on upper level.
            return;
        }
        mHandlingEvent = true;
        sync();
        mHandlingEvent = false;
}

private void sync() {
        LifecycleOwner lifecycleOwner = mLifecycleOwner.get();
        if (lifecycleOwner == null) {
            return;
        }
  //只要还没有全部同步，那就继续发送事件 ，循环 遍历所有被观察者
        while (!isSynced()) {
            mNewEventOccurred = false;
            if (mState.compareTo(mObserverMap.eldest().getValue().mState) < 0) {// 发送事件
                backwardPass(lifecycleOwner);
            }
            Entry<LifecycleObserver, ObserverWithState> newest = mObserverMap.newest();
            if (!mNewEventOccurred && newest != null
                    && mState.compareTo(newest.getValue().mState) > 0) {
              // 发送事件
                forwardPass(lifecycleOwner);
            }
        }
        mNewEventOccurred = false;
    }

// 状态是否同步
    private boolean isSynced() {
        if (mObserverMap.size() == 0) {
            return true;
        }
        State eldestObserverState = mObserverMap.eldest().getValue().mState;
        State newestObserverState = mObserverMap.newest().getValue().mState;
        return eldestObserverState == newestObserverState && mState == newestObserverState;
    }

// 循环 遍历 被观察者 中 所有观察者 发送事件
private void forwardPass(LifecycleOwner lifecycleOwner) {
        Iterator<Entry<LifecycleObserver, ObserverWithState>> ascendingIterator =
                mObserverMap.iteratorWithAdditions();
        while (ascendingIterator.hasNext() && !mNewEventOccurred) {
            Entry<LifecycleObserver, ObserverWithState> entry = ascendingIterator.next();
            ObserverWithState observer = entry.getValue();
            while ((observer.mState.compareTo(mState) < 0 && !mNewEventOccurred
                    && mObserverMap.contains(entry.getKey()))) {
                pushParentState(observer.mState);
                observer.dispatchEvent(lifecycleOwner, upEvent(observer.mState));
                popParentState();
            }
        }
    }
```

```java
//ObserverWithState
static class ObserverWithState {
        State mState;
        GenericLifecycleObserver mLifecycleObserver;

        ObserverWithState(LifecycleObserver observer, State initialState) {
          //这里 创建了一个GenericLifeCycleObserver
            mLifecycleObserver = Lifecycling.getCallback(observer);
            mState = initialState;
        }

        void dispatchEvent(LifecycleOwner owner, Event event) {
            State newState = getStateAfter(event);
            mState = min(mState, newState);
          //调用被观察者，发送事件
            mLifecycleObserver.onStateChanged(owner, event);
            mState = newState;
        }
    }
```

在`ObserverWithState`的构造方法中，创建了一个GenericLifeCycleObserver：

```java
// Lifecycling.java
@NonNull
    static GenericLifecycleObserver getCallback(Object object) {
        if (object instanceof FullLifecycleObserver) {
            return new FullLifecycleObserverAdapter((FullLifecycleObserver) object);
        }

        if (object instanceof GenericLifecycleObserver) {
            return (GenericLifecycleObserver) object;
        }

        final Class<?> klass = object.getClass();
        int type = getObserverConstructorType(klass);
        if (type == GENERATED_CALLBACK) {
            List<Constructor<? extends GeneratedAdapter>> constructors =
                    sClassToAdapters.get(klass);
            if (constructors.size() == 1) {
                GeneratedAdapter generatedAdapter = createGeneratedAdapter(
                        constructors.get(0), object);
                return new SingleGeneratedAdapterObserver(generatedAdapter);
            }
            GeneratedAdapter[] adapters = new GeneratedAdapter[constructors.size()];
            for (int i = 0; i < constructors.size(); i++) {
                adapters[i] = createGeneratedAdapter(constructors.get(i), object);
            }
            return new CompositeGeneratedAdaptersObserver(adapters);
        }
        return new ReflectiveGenericLifecycleObserver(object);
    }
```

大概意思就是创建了一个`GenericLifecycleObserver`的接口实现类，

```java
//GenericLifecycleObserver.java
public interface GenericLifecycleObserver extends LifecycleObserver {
    void onStateChanged(LifecycleOwner source, Lifecycle.Event event);
}

```

参考一下`FullLifecycleObserverAdapter`中：

```java
//FullLifecycleObserverAdapter.java
class FullLifecycleObserverAdapter implements GenericLifecycleObserver {

    private final FullLifecycleObserver mObserver;

    FullLifecycleObserverAdapter(FullLifecycleObserver observer) {
        mObserver = observer;
    }

    @Override
    public void onStateChanged(LifecycleOwner source, Lifecycle.Event event) {
        switch (event) {
            case ON_CREATE:
                mObserver.onCreate(source);
                break;
            case ON_START:
                mObserver.onStart(source);
                break;
            case ON_RESUME:
                mObserver.onResume(source);
                break;
            case ON_PAUSE:
                mObserver.onPause(source);
                break;
            case ON_STOP:
                mObserver.onStop(source);
                break;
            case ON_DESTROY:
                mObserver.onDestroy(source);
                break;
            case ON_ANY:
                throw new IllegalArgumentException("ON_ANY must not been send by anybody");
        }
    }
}
```

   





```java
//FullLifecycleObserver.java

interface FullLifecycleObserver extends LifecycleObserver {

    void onCreate(LifecycleOwner owner);

    void onStart(LifecycleOwner owner);

    void onResume(LifecycleOwner owner);

    void onPause(LifecycleOwner owner);

    void onStop(LifecycleOwner owner);

    void onDestroy(LifecycleOwner owner);
}
```

其实是当状态改变时候，观察者的具体操作，如`FullLifecycleObserver`中指定调用状态对应的方法。



**注解执行：** 

基于`ReflectiveGenericLifecycleObserver`反射执行



```java
//ClassesInfoCache.java

class ClassesInfoCache {
// 此处是单例
    static ClassesInfoCache sInstance = new ClassesInfoCache();

    private static final int CALL_TYPE_NO_ARG = 0;
    private static final int CALL_TYPE_PROVIDER = 1;
    private static final int CALL_TYPE_PROVIDER_WITH_EVENT = 2;

    private final Map<Class, CallbackInfo> mCallbackMap = new HashMap<>();
    private final Map<Class, Boolean> mHasLifecycleMethods = new HashMap<>();

    boolean hasLifecycleMethods(Class klass) {
        if (mHasLifecycleMethods.containsKey(klass)) {
            return mHasLifecycleMethods.get(klass);
        }

        Method[] methods = getDeclaredMethods(klass);
        for (Method method : methods) {
            OnLifecycleEvent annotation = method.getAnnotation(OnLifecycleEvent.class);
            if (annotation != null) {
                // Optimization for reflection, we know that this method is called
                // when there is no generated adapter. But there are methods with @OnLifecycleEvent
              // 这里 因为是注解，那就是使用ReflectiveGenericLifecycleObserver
                // so we know that will use ReflectiveGenericLifecycleObserver,
                // so we createInfo in advance.
                // CreateInfo always initialize mHasLifecycleMethods for a class, so we don't do it
                // here.
                createInfo(klass, methods);
                return true;
            }
        }
        mHasLifecycleMethods.put(klass, false);
        return false;
    }
  ...
}
```



```java
class ReflectiveGenericLifecycleObserver implements GenericLifecycleObserver {
    private final Object mWrapped;
    private final CallbackInfo mInfo;

    ReflectiveGenericLifecycleObserver(Object wrapped) {
        mWrapped = wrapped;
        mInfo = ClassesInfoCache.sInstance.getInfo(mWrapped.getClass());
    }

    @Override
    public void onStateChanged(LifecycleOwner source, Event event) {
      //反射执行对应方法
        mInfo.invokeCallbacks(source, event, mWrapped);
    }
}
```



```java
//CallbackInfo

static class CallbackInfo {
    final Map<Lifecycle.Event, List<MethodReference>> mEventToHandlers;
  //key是监听方法的List,value 是事件
    final Map<MethodReference, Lifecycle.Event> mHandlerToEvent;

    CallbackInfo(Map<MethodReference, Lifecycle.Event> handlerToEvent) {
        mHandlerToEvent = handlerToEvent;
        mEventToHandlers = new HashMap<>();
      // 拿出方法和事件，K变成事件，v变成方法数组
        for (Map.Entry<MethodReference, Lifecycle.Event> entry : handlerToEvent.entrySet()) {
            Lifecycle.Event event = entry.getValue();
            List<MethodReference> methodReferences = mEventToHandlers.get(event);
            if (methodReferences == null) {
                methodReferences = new ArrayList<>();
              //1.  这里，给这个map添加数据
                mEventToHandlers.put(event, methodReferences);
            }
            methodReferences.add(entry.getKey());
        }
    }
//调用的是这里反射执行方法
    @SuppressWarnings("ConstantConditions")
    void invokeCallbacks(LifecycleOwner source, Lifecycle.Event event, Object target) {
      // 2. 注意 mEventToHandlers.get(event) 这个参数
        invokeMethodsForEvent(mEventToHandlers.get(event), source, event, target);
        invokeMethodsForEvent(mEventToHandlers.get(Lifecycle.Event.ON_ANY), source, event,
                target);
    }

    private static void invokeMethodsForEvent(List<MethodReference> handlers,
            LifecycleOwner source, Lifecycle.Event event, Object mWrapped) {
        if (handlers != null) {
            for (int i = handlers.size() - 1; i >= 0; i--) {
                handlers.get(i).invokeCallback(source, event, mWrapped);
            }
        }
    }
}
```



上述代码中

1. 给map 添加数据，其实是事件对应的所有方法的map，key是事件，value是监听方法的List
2. 获取对应事件的所有方法，分别执行

```java
static class MethodReference {
        final int mCallType;
        final Method mMethod;

        MethodReference(int callType, Method method) {
            mCallType = callType;
            mMethod = method;
            mMethod.setAccessible(true);
        }
// 执行方法
        void invokeCallback(LifecycleOwner source, Lifecycle.Event event, Object target) {
            //noinspection TryWithIdenticalCatches
            try {
                switch (mCallType) {
                    case CALL_TYPE_NO_ARG:
                        mMethod.invoke(target);
                        break;
                    case CALL_TYPE_PROVIDER:
                        mMethod.invoke(target, source);
                        break;
                    case CALL_TYPE_PROVIDER_WITH_EVENT:
                        mMethod.invoke(target, source, event);
                        break;
                }
         ...
        }
```

找到了执行方法，那么注解的事件对应的方法列表是如何添加进来的，也就是说`CallbackInfo`是什么时候初始化的。追踪代码可发现，获取kv起点是在`Lifecycling`中调用的，

```java
//Lifecycling.java

@NonNull
    static GenericLifecycleObserver getCallback(Object object) {
       ...
        int type = getObserverConstructorType(klass);
       ...
    }

 private static int getObserverConstructorType(Class<?> klass) {
      ...
        int type = resolveObserverCallbackType(klass);
      ...
    }

private static int resolveObserverCallbackType(Class<?> klass) {
        ...
       // 这里是起点
        boolean hasLifecycleMethods = ClassesInfoCache.sInstance.hasLifecycleMethods(klass);
        ...
        }
```

后续就是在`ClassesInfoCache`中遍历类中的所有方法，找到被事件注解的方法，放到map中的操作了。最后在执行的时候，从map中取出事件对应的方法数组，遍历执行。



## Lifecycles 的最佳实践

> 本小节内容节选自《[译] Architecture Components 之 Handling Lifecycles》
> 作者：zly394
> 链接：https://juejin.im/post/5937e1c8570c35005b7b262a

- 保持 UI 控制器（Activity 和 Fragment）尽可能的精简。它们不应该试图去获取它们所需的数据；相反，要用 ViewModel来获取，并且观察 LiveData将数据变化反映到视图中。


- 尝试编写数据驱动（data-driven）的 UI，即 UI 控制器的责任是在数据改变时更新视图或者将用户的操作通知给 ViewModel。


- 将数据逻辑放到 ViewModel 类中。ViewModel 应该作为 UI 控制器和应用程序其它部分的连接服务。注意：不是由 ViewModel 负责获取数据（例如：从网络获取）。相反，ViewModel 调用相应的组件获取数据，然后将数据获取结果提供给 UI 控制器。


- 使用Data Binding来保持视图和 UI 控制器之间的接口干净。这样可以让视图更具声明性，并且尽可能减少在 Activity 和 Fragment 中编写更新代码。如果你喜欢在 Java 中执行该操作，请使用像Butter Knife 这样的库来避免使用样板代码并进行更好的抽象化。


- 如果 UI 很复杂，可以考虑创建一个 Presenter 类来处理 UI 的修改。虽然通常这样做不是必要的，但可能会让 UI 更容易测试。


- 不要在 ViewModel 中引用View或者 Activity的 context。因为如果ViewModel存活的比 Activity 时间长（在配置更改的情况下），Activity 将会被泄漏并且无法被正确的回收。



## FullLifecycleObserver、DefaultLifecycleObserver

除了注解，还有一种是实现`DefaultLifecycleObserver`接口，其中已经写好了各个事件对应的方法，且有默认空实现，

DefaultLifecycleObserver 类中的文档提到，

```java
/**
 * Callback interface for listening to {@link LifecycleOwner} state changes.
 * <p>
 * If you use Java 8 language, <b>always</b> prefer it over annotations.
 */
```

如果你使用了 Java8，那么就推荐使用 DefaultLifecycleObserver。

> 我使用的是：
>
> ```java
> implementation "android.arch.lifecycle:extensions:1.1.1"
> ```
>
> 其中没有DefaultLifecycleObserver，可能已经取消了，但是我找到了FullLifecycleObserver，有相同作用

```java
interface FullLifecycleObserver extends LifecycleObserver {

    void onCreate(LifecycleOwner owner);

    void onStart(LifecycleOwner owner);

    void onResume(LifecycleOwner owner);

    void onPause(LifecycleOwner owner);

    void onStop(LifecycleOwner owner);

    void onDestroy(LifecycleOwner owner);
}
```



```java
class FullLifecycleObserverAdapter implements GenericLifecycleObserver {

    private final FullLifecycleObserver mObserver;

    FullLifecycleObserverAdapter(FullLifecycleObserver observer) {
        mObserver = observer;
    }

    @Override
    public void onStateChanged(LifecycleOwner source, Lifecycle.Event event) {
        switch (event) {
            case ON_CREATE:
                mObserver.onCreate(source);
                break;
            case ON_START:
                mObserver.onStart(source);
                break;
            case ON_RESUME:
                mObserver.onResume(source);
                break;
            case ON_PAUSE:
                mObserver.onPause(source);
                break;
            case ON_STOP:
                mObserver.onStop(source);
                break;
            case ON_DESTROY:
                mObserver.onDestroy(source);
                break;
            case ON_ANY:
                throw new IllegalArgumentException("ON_ANY must not been send by anybody");
        }
    }
}
```





[参考文章]:     https://blog.csdn.net/mq2553299/article/details/79029657
[参考文章]:https://juejin.im/post/5c90d955f265da60e926783d







\```java

//CallbackInfo



static class CallbackInfo {

final Map<Lifecycle.Event, List<MethodReference>> mEventToHandlers;

//key是监听方法的List,value 是事件

final Map<MethodReference, Lifecycle.Event> mHandlerToEvent;



CallbackInfo(Map<MethodReference, Lifecycle.Event> handlerToEvent) {

mHandlerToEvent = handlerToEvent;

mEventToHandlers = new HashMap<>();

// 拿出方法和事件，K变成事件，v变成方法数组

for (Map.Entry<MethodReference, Lifecycle.Event> entry : handlerToEvent.entrySet()) {

Lifecycle.Event event = entry.getValue();

List<MethodReference> methodReferences = mEventToHandlers.get(event);

if (methodReferences == null) {

methodReferences = new ArrayList<>();

//1. 这里，给这个map添加数据

mEventToHandlers.put(event, methodReferences);

}

methodReferences.add(entry.getKey());

}

}

//调用的是这里反射执行方法

@SuppressWarnings("ConstantConditions")

void invokeCallbacks(LifecycleOwner source, Lifecycle.Event event, Object target) {

// 2. 注意 mEventToHandlers.get(event) 这个参数

invokeMethodsForEvent(mEventToHandlers.get(event), source, event, target);

invokeMethodsForEvent(mEventToHandlers.get(Lifecycle.Event.ON_ANY), source, event,

target);

}



private static void invokeMethodsForEvent(List<MethodReference> handlers,

LifecycleOwner source, Lifecycle.Event event, Object mWrapped) {

if (handlers != null) {

for (int i = handlers.size() - 1; i >= 0; i--) {

handlers.get(i).invokeCallback(source, event, mWrapped);

}

}

}

}

\```







上述代码中



\1. 给map 添加数据，其实是事件对应的所有方法的map，key是事件，value是监听方法的List

\2. 获取对应事件的所有方法，分别执行



\```java

static class MethodReference {

final int mCallType;

final Method mMethod;



MethodReference(int callType, Method method) {

mCallType = callType;

mMethod = method;

mMethod.setAccessible(true);

}

// 执行方法

void invokeCallback(LifecycleOwner source, Lifecycle.Event event, Object target) {

//noinspection TryWithIdenticalCatches

try {

switch (mCallType) {

case CALL_TYPE_NO_ARG:

mMethod.invoke(target);

break;

case CALL_TYPE_PROVIDER:

mMethod.invoke(target, source);

break;

case CALL_TYPE_PROVIDER_WITH_EVENT:

mMethod.invoke(target, source, event);

break;

}

...

}

\```



找到了执行方法，那么注解的事件对应的方法列表是如何添加进来的，也就是说`CallbackInfo`是什么时候初始化的。追踪代码可发现，获取kv起点是在`Lifecycling`中调用的，



\```java

//Lifecycling.java



@NonNull

static GenericLifecycleObserver getCallback(Object object) {

...

int type = getObserverConstructorType(klass);

...

}



private static int getObserverConstructorType(Class<?> klass) {

...

int type = resolveObserverCallbackType(klass);

...

}



private static int resolveObserverCallbackType(Class<?> klass) {

...

// 这里是起点

boolean hasLifecycleMethods = ClassesInfoCache.sInstance.hasLifecycleMethods(klass);

...

}

\```



后续就是在`ClassesInfoCache`中遍历类中的所有方法，找到被事件注解的方法，放到map中的操作了。最后在执行的时候，从map中取出事件对应的方法数组，遍历执行。







\## Lifecycles 的最佳实践



\> 本小节内容节选自《[译] Architecture Components 之 Handling Lifecycles》

\> 作者：zly394

\> 链接：https://juejin.im/post/5937e1c8570c35005b7b262a



\- 保持 UI 控制器（Activity 和 Fragment）尽可能的精简。它们不应该试图去获取它们所需的数据；相反，要用 ViewModel来获取，并且观察 LiveData将数据变化反映到视图中。





\- 尝试编写数据驱动（data-driven）的 UI，即 UI 控制器的责任是在数据改变时更新视图或者将用户的操作通知给 ViewModel。





\- 将数据逻辑放到 ViewModel 类中。ViewModel 应该作为 UI 控制器和应用程序其它部分的连接服务。注意：不是由 ViewModel 负责获取数据（例如：从网络获取）。相反，ViewModel 调用相应的组件获取数据，然后将数据获取结果提供给 UI 控制器。





\- 使用Data Binding来保持视图和 UI 控制器之间的接口干净。这样可以让视图更具声明性，并且尽可能减少在 Activity 和 Fragment 中编写更新代码。如果你喜欢在 Java 中执行该操作，请使用像Butter Knife 这样的库来避免使用样板代码并进行更好的抽象化。





\- 如果 UI 很复杂，可以考虑创建一个 Presenter 类来处理 UI 的修改。虽然通常这样做不是必要的，但可能会让 UI 更容易测试。





\- 不要在 ViewModel 中引用View或者 Activity的 context。因为如果ViewModel存活的比 Activity 时间长（在配置更改的情况下），Activity 将会被泄漏并且无法被正确的回收。







\## FullLifecycleObserver、DefaultLifecycleObserver



除了注解，还有一种是实现`DefaultLifecycleObserver`接口，其中已经写好了各个事件对应的方法，且有默认空实现，



DefaultLifecycleObserver 类中的文档提到，



\```java

/**

\* Callback interface for listening to {@link LifecycleOwner} state changes.

\* <p>

\* If you use Java 8 language, <b>always</b> prefer it over annotations.

*/

\```



如果你使用了 Java8，那么就推荐使用 DefaultLifecycleObserver。



\> 我使用的是：

\>

\> ```java

\> implementation "android.arch.lifecycle:extensions:1.1.1"

\> ```

\>

\> 其中没有DefaultLifecycleObserver，可能已经取消了，但是我找到了FullLifecycleObserver，有相同作用



\```java

interface FullLifecycleObserver extends LifecycleObserver {



void onCreate(LifecycleOwner owner);



void onStart(LifecycleOwner owner);



void onResume(LifecycleOwner owner);



void onPause(LifecycleOwner owner);



void onStop(LifecycleOwner owner);



void onDestroy(LifecycleOwner owner);

}

\```







\```java

class FullLifecycleObserverAdapter implements GenericLifecycleObserver {



private final FullLifecycleObserver mObserver;



FullLifecycleObserverAdapter(FullLifecycleObserver observer) {

mObserver = observer;

}



@Override

public void onStateChanged(LifecycleOwner source, Lifecycle.Event event) {

switch (event) {

case ON_CREATE:

mObserver.onCreate(source);

break;

case ON_START:

mObserver.onStart(source);

break;

case ON_RESUME:

mObserver.onResume(source);

break;

case ON_PAUSE:

mObserver.onPause(source);

break;

case ON_STOP:

mObserver.onStop(source);

break;

case ON_DESTROY:

mObserver.onDestroy(source);

break;

case ON_ANY:

throw new IllegalArgumentException("ON_ANY must not been send by anybody");

}

}

}

\```











[参考文章]: https://blog.csdn.net/mq2553299/article/details/79029657
[参考文章]: https://juejin.im/post/5c90d955f265da60e926783d

