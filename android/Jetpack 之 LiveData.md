# Jetpack 之 LiveData

[TOC]



## 概述

- LiveData 是一个持有数据的类，它持有的数据是可以被观察者订阅的，当数据被修改时就会通知观察者。观察者可以是 Activity、Fragment、Service 等。
- LiveData 能够感知观察者的生命周期，只有当观察者处于激活状态（STARTED、RESUMED）才会接收到数据更新的通知，在未激活时会自动解注册观察者，以减少内存泄漏。
- 使用 LiveData 保存数据时，由于数据和组件是分离的，当组件重建时可以保证数据不会丢失。

## 优点

- 确保 UI 界面始终和数据状态保持一致。
- 没有内存泄漏，观察者绑定到 Lifecycle 对象并在其相关生命周期 destroyed 后自行解除绑定。
- 不会因为 Activity 停止了而奔溃，如 Activity finish 了，它就不会收到任何 LiveData 事件了。
- UI 组件只需观察相关数据，不需要停止或恢复观察，LiveData 会自动管理这些操作，因为 LiveData 可以感知生命周期状态的更改。
- 在生命周期从非激活状态变为激活状态，始终保持最新数据，如后台 Activity 在返回到前台后可以立即收到最新数据。
- 当配置发生更改（如屏幕旋转）而重建 Activity / Fragment，它会立即收到最新的可用数据。
- LiveData 很适合用于组件（Activity / Fragment）之间的通信。

## 使用

### 导入依赖

[添加相关依赖](https://developer.android.com/topic/libraries/architecture/adding-components)

LiveData 有两种使用方式，结合 ViewModel 使用以及直接继承 LiveData 类。

```groovy
// ViewModel and LiveData
implementation "android.arch.lifecycle:extensions:1.1.0"
// alternatively, just ViewModel
implementation "android.arch.lifecycle:viewmodel:1.1.0"
// alternatively, just LiveData
implementation "android.arch.lifecycle:livedata:1.1.0"
```



### 结合 ViewModel 使用

LiveData 是一个抽象类，它的实现子类有 MutableLiveData ，MediatorLiveData。在实际使用中，用得比较多的是 MutableLiveData。常常结合 ViewModel 一起使用。

```java
public class TestViewModel extends ViewModel {

    private MutableLiveData<String> mNameEvent = new MutableLiveData<>();

    public MutableLiveData<String> getNameEvent() {
        return mNameEvent;
    }

}
```

在Activity中创建ViewModel，监听ViewModel中的数据变化

```java
mTestViewModel = ViewModelProviders.of(this).get(TestViewModel.class);
MutableLiveData<String> nameEvent = mTestViewModel.getNameEvent();
nameEvent.observe(this, new Observer<String>() {
    @Override
    public void onChanged(@Nullable String s) {
        Log.i(TAG, "onChanged: s = " + s);
        mTvName.setText(s);
    }
});
```



<font color = "#ff00e0">**Activity、fragment 数据传递：**</font>

对应activity、viewModel 获取到的对象是相同的，所以可用来Activity、fragment通信

fragment中：

```java
nameEvent = ViewModelProviders.of(getActivity()).get(HomeFragmentViewModel.class);
nameEvent.observe(this, new Observer<String>() {
    @Override
    public void onChanged(@Nullable String s) {
       ....
    }
});
```



<font color = "#ff0000">**ViewModel 携带参数：**</font>

		同样是调用 ViewModelProvider of(@NonNull Fragment fragment, @Nullable Factory factory) 方法，只不过，需要多传递一个 factory 参数。	

**做法：**

*实现 Factory 接口，重写 create 方法，在create 方法里面调用相应的构造函数，返回相应的实例。*

```java
public class TestViewModel extends ViewModel {

    private final String mKey;
    private MutableLiveData<String> mNameEvent = new MutableLiveData<>();

    public MutableLiveData<String> getNameEvent() {
        return mNameEvent;
    }

    public TestViewModel(String key) {
        mKey = key;
    }

    public static class Factory implements ViewModelProvider.Factory {
        private String mKey;

        public Factory(String key) {
            mKey = key;
        }

        @Override
        public <T extends ViewModel> T create(Class<T> modelClass) {
            return (T) new TestViewModel(mKey);
        }
    }

    public String getKey() {
        return mKey;
    }
}
```

```java
ViewModelProviders.of(this, new TestViewModel.Factory(mkey)).get(TestViewModel.class)
```



### 直接继承 LiveData 类

以下代码场景：在 Activity 中监听 Wifi 信号强度。

```kotlin
class WifiLiveData private constructor(context: Context) : LiveData<Int>() {

    private var mContext: WeakReference<Context> = WeakReference(context)

    companion object {

        private var instance: WifiLiveData? = null

        fun getInstance(context: Context): WifiLiveData {
            if (instance == null) {
                instance = WifiLiveData(context)
            }
            return instance!!
        }
    }

    override fun onActive() {
        super.onActive()
        registerReceiver()
    }

    override fun onInactive() {
        super.onInactive()
        unregisterReceiver()
    }

    /**
     * 注册广播，监听 Wifi 信号强度
     */
    private fun registerReceiver() {
        val intentFilter = IntentFilter()
        intentFilter.addAction(WifiManager.RSSI_CHANGED_ACTION)
        mContext.get()!!.registerReceiver(mReceiver, intentFilter)
    }

    /**
     * 注销广播
     */
    private fun unregisterReceiver() {
        mContext.get()!!.unregisterReceiver(mReceiver)
    }

    private val mReceiver = object : BroadcastReceiver() {

        override fun onReceive(context: Context?, intent: Intent) {
            when (intent.action) {
                WifiManager.RSSI_CHANGED_ACTION -> getWifiLevel()
            }
        }
    }

    private fun getWifiLevel() {
        val wifiManager = mContext.get()!!.applicationContext.getSystemService(android.content.Context.WIFI_SERVICE) as WifiManager
        val wifiInfo = wifiManager.connectionInfo
        val level = wifiInfo.rssi

        instance!!.value = level // 发送 Wifi 的信号强度给观察者
    }
}

```

```kotlin
class LiveDataActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_live_data)

        withExtendsLiveDataTest()
    }

    /**
     * 直接继承 LiveData 类
     */
    private fun withExtendsLiveDataTest() {
        WifiLiveData.getInstance(this).observe(this, Observer {
            Log.e("LiveDataActivity", it.toString()) // 观察者收到数据更新的通知，打印 Wifi 信号强度
        })
    }
}
	
```

> 当组件（Activity）处于激活状态（onActive）时注册广播，处于非激活状态（onInactive）时注销广播。



## 源码解析

### observe 注册流程

LiveData 通过 `observe()` 方法将被观察者 `LifecycleOwner` (Activity / Fragment) 和观察者 Observer 关联起来。

```java
LiveData.observe(LifecycleOwner owner , Observer<T> observer)
```

LiveData源码：

```java
public void observe(@NonNull LifecycleOwner owner, @NonNull Observer<T> observer) {
    if (owner.getLifecycle().getCurrentState() == DESTROYED) {
        // 若 LifecycleOwner 处于 DESTROYED 状态，则返回
        return;
    }

    // LifecycleBoundObserver 把 LifecycleOwner 对象和 Observer 对象包装在一起
    LifecycleBoundObserver wrapper = new LifecycleBoundObserver(owner, observer);

    // mObservers（类似 Map 的容器）的 putIfAbsent() 方法用于判断容器中的 observer（key）
    // 是否已有 wrapper（value）与之关联
    // 若已关联则直接返回关联值，否则关联后再返回 wrapper
    ObserverWrapper existing = mObservers.putIfAbsent(observer, wrapper);

    if (existing != null && !existing.isAttachedTo(owner)) {
        throw new IllegalArgumentException("Cannot add the same observer"
                + " with different lifecycles");
    }
    if (existing != null) {
        return;
    }

    // 由于 LifecycleBoundObserver 实现了 GenericLifecycleObserver 接口，而 GenericLifecycleObserver 又
    // 继承了 LifecycleObserver，所以 LifecycleBoundObserver 本质是一个 LifecycleObserver
    // 此处属于注册过程， Lifecycle 添加观察者 LifecycleObserver
    owner.getLifecycle().addObserver(wrapper);
}
```



### 感知生命周期变化

由上可知，`LifecycleBoundObserver`（LiveData 的内部类）是观察者，以下具体分析 `LifecycleBoundObserver` 的实现过程。

```java
class LifecycleBoundObserver extends ObserverWrapper implements GenericLifecycleObserver {
    @NonNull final LifecycleOwner mOwner;

    LifecycleBoundObserver(@NonNull LifecycleOwner owner, Observer<T> observer) {
        super(observer); // 保存 Observer
        mOwner = owner;  // 保存 LifecycleOwner
    }

    @Override
    boolean shouldBeActive() {
        // 判断是否处于激活状态
        return mOwner.getLifecycle().getCurrentState().isAtLeast(STARTED);
    }


    @Override
    public void onStateChanged(LifecycleOwner source, Lifecycle.Event event) {
        // 若 Lifecycle 处于 DESTROYED 状态，则移除 Observer 对象
        if (mOwner.getLifecycle().getCurrentState() == DESTROYED) {
            // 移除观察者，在这个方法中会移除生命周期监听并且回调 activeStateChanged() 方法
            removeObserver(mObserver);
            return;
        }
        // 若处于激活状态，则调用 activeStateChanged() 方法
        activeStateChanged(shouldBeActive());
    }

    @Override
    boolean isAttachedTo(LifecycleOwner owner) {
        return mOwner == owner;
    }

    @Override
    void detachObserver() {
        mOwner.getLifecycle().removeObserver(this);
    }
}
```

当组件（Activity / Fragment）的生命周期发生改变时，onStateChanged() 方法将会被调用。若当前处于 DESTROYED 状态，则会移除观察者；若当前处于激活状态，则会调用 activeStateChanged() 方法。activeStateChanged() 方法位于父类 ObserverWrapper 中。

```java
void activeStateChanged(boolean newActive) {
    // 若新旧状态一致，则返回
    if (newActive == mActive) {
        return;
    }
    // immediately set active state, so we'd never dispatch anything to inactive owner
    mActive = newActive;
    boolean wasInactive = LiveData.this.mActiveCount == 0;
    LiveData.this.mActiveCount += mActive ? 1 : -1;
    if (wasInactive && mActive) { // 激活状态的 observer 个数从 0 到 1
        onActive(); // 空实现，一般让子类去重写
    }
    if (LiveData.this.mActiveCount == 0 && !mActive) { // 激活状态的 observer 个数从 1 到 0
        onInactive();  // 空实现，一般让子类去重写
    }
    if (mActive) { // 激活状态，向观察者发送 LiveData 的值
        dispatchingValue(this);
    }
}
```

分发Value ：`dispatchingValue`

```java
private void dispatchingValue(@Nullable ObserverWrapper initiator) {
    // ...
    do {
        mDispatchInvalidated = false;
        if (initiator != null) {
            considerNotify(initiator);
            initiator = null;
        } else {
            // 循环遍历 mObservers 这个 map , 向每一个观察者都发送新的数据
            for (Iterator<Map.Entry<Observer<T>, ObserverWrapper>> iterator =
                    mObservers.iteratorWithAdditions(); iterator.hasNext(); ) {
                considerNotify(iterator.next().getValue());
                if (mDispatchInvalidated) {
                    break;
                }
            }
        }
    } while (mDispatchInvalidated);
    // ...
}
```

`considerNotify` 发送事件：

```java
private void considerNotify(ObserverWrapper observer) {
    // ...
    observer.mObserver.onChanged((T) mData);
}
```

上面的 mObserver 正是调用 observe() 方法时传入的观察者。

### 更新数据方式

##### `setValue` 

```java
 @MainThread
    protected void setValue(T value) {
        assertMainThread("setValue");
        mVersion++;
        mData = value;
        dispatchingValue(null);
    }
```

`dispatchingValue`如上小结分析的value分发方法

##### `postValue` 

```java
 protected void postValue(T value) {
        boolean postTask;
        synchronized (mDataLock) {
            postTask = mPendingData == NOT_SET;
            mPendingData = value;
        }
        if (!postTask) {
            return;
        }
        ArchTaskExecutor.getInstance().postToMainThread(mPostValueRunnable);
    }
```

`ArchTaskExecutor` 中`postToMainThread`方法：

```java
 @Override
    public void postToMainThread(Runnable runnable) {
      //mDelegate是 DefaultTaskExecutor 对象
        mDelegate.postToMainThread(runnable);
    }
```

 `DefaultTaskExecutor` （是`TaskExecutor`的实现类）代码：

```java
@RestrictTo(RestrictTo.Scope.LIBRARY_GROUP)
public class DefaultTaskExecutor extends TaskExecutor {
    private final Object mLock = new Object();
    private ExecutorService mDiskIO = Executors.newFixedThreadPool(2);

    @Nullable
    private volatile Handler mMainHandler;

    @Override
    public void executeOnDiskIO(Runnable runnable) {
        mDiskIO.execute(runnable);
    }

  //切换到主线程执行
    @Override
    public void postToMainThread(Runnable runnable) {
        if (mMainHandler == null) {
            synchronized (mLock) {
                if (mMainHandler == null) {
                    mMainHandler = new Handler(Looper.getMainLooper());
                }
            }
        }
        //noinspection ConstantConditions
        mMainHandler.post(runnable);
    }

    @Override
    public boolean isMainThread() {
        return Looper.getMainLooper().getThread() == Thread.currentThread();
    }
}
```

##### <font color = "#ff00e0">两种更新方式的区别：</font>

- `setValue`：更新数据是在调用`setValue`方法所在线程更新

- `postValue`：更新数据是在主线程更新