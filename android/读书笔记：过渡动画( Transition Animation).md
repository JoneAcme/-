# 过渡动画( Transition Animation)

## 简介

​		过渡动画是在 Android4.4引入的新的动画框架,它**本质上还是属性动画**,只不过是对属性动画做了一层封装,方便开发者实现 Activity或者View的过渡动画效果。和属性动画相比, 过渡动画最大的不同是需要为动画前后准备不同的布局,并通过对应的AI实现两个布局的过渡动画,而属性动画只需要一个布局文件。



## 基本概念

- Scene:定义了页面的当前状态信息, Scene的实例化一般通过静态工厂方法实现。
  public static Scene getSceneForLayout(Viewgroup sceneroot, int layoutid Context context)

- Transition:定义了界面之间切换的动画信息,在使用 Transition Manager时没有指定使用哪个 Transition,那么会使用默认的 Auto Transition,源码如下。可以看出 Autotransition 的动画效果就是先隐藏对象变透明,然后移动指定的对象,最后显示出来。

  ```java
  public class AutoTransition extends TransitionSet {
  
      public AutoTransition() {
          init();
      }
  
      public AutoTransition(Context context, AttributeSet attrs) {
          super(context, attrs);
          init();
      }
  
      private void init() {
          setOrdering(ORDERING_SEQUENTIAL);
          addTransition(new Fade(Fade.OUT)).
                  addTransition(new ChangeBounds()).
                  addTransition(new Fade(Fade.IN));
      }
  }
  ```

- TransitionManager:控制 Scene之间切换的控制器,切换常用的方法有以下两个,其中的`sDefaultTransition`就是前面说的 Auto Transition的实例

  ```java
  
      public void transitionTo(Scene scene) {
          // Auto transition if there is no transition declared for the Scene, but there is
          // a root or parent view
          changeScene(scene, getTransition(scene));
      }
  
      public static void go(Scene scene) {
          changeScene(scene, sDefaultTransition);
      }
  
      public static void go(Scene scene, Transition transition) {
          changeScene(scene, transition);
      }
  ```

  

## 使用方式

1. res 资源文件目录下新建 transition ，用于存放TransitionManager和TransitionSet

2. 首先定义同一个页面的两个布局,分别是动画前的布局和动画后的布局,将其命名为 layout_translation_before.xml和 layout_translation_after.xml，这两个布局文件的根布局具有相同的 id值,动画前的布局文件内容如下。

    `layout_translation_before.xml`

   ```xml
   <?xml version="1.0" encoding="utf-8"?>
   <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
           android:layout_width="match_parent"
           android:layout_height="match_parent"
           android:id="@+id/scene"
           android:orientation="vertical">
   
       <Button
               android:id="@+id/btn"
               android:layout_width="match_parent"
               android:layout_height="wrap_content"
               android:text="this is button before" />
   
       <TextView
               android:id="@+id/textview"
               android:layout_width="match_parent"
               android:layout_height="wrap_content"
               android:gravity="center"
               android:text="this is textView before" />
   </LinearLayout>
   ```

    `layout_translation_after.xml`

   ```xml
   <?xml version="1.0" encoding="utf-8"?>
   <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
           android:id="@+id/scene"
           android:layout_width="match_parent"
           android:layout_height="match_parent"
           android:gravity="center"
           android:orientation="vertical">
   
   
       <TextView
               android:id="@+id/textview"
               android:layout_width="match_parent"
               android:layout_height="wrap_content"
               android:gravity="center"
               android:text="this is textView after" />
   </LinearLayout>
   ```

   **注意**： 要想使View在切换时候有动画，那么对该View 在两个布局中要有相同的 ID，

3. 定义TransitionManager

   ```xml
   <?xml version="1.0" encoding="utf-8"?>
   <transitionManager xmlns:android="http://schemas.android.com/apk/res/android">
   
       <transition
               android:fromScene="@layout/layout_translation_before"
               android:toScene="@layout/layout_translation_after"
               android:transition="@transition/trans_set" />
   
       <transition
               android:fromScene="@layout/layout_translation_after"
               android:toScene="@layout/layout_translation_before"
               android:transition="@transition/trans_set" />
   
   </transitionManager>
   ```

   其中 transition 属性指定的为 移动过程中要执行的动画 资源文件：

   `transition/trans_set.xml`

   ```xml
   <?xml version="1.0" encoding="utf-8"?>
   <transitionSet xmlns:android="http://schemas.android.com/apk/res/android">
   
       <fade
               android:duration="1000"
               android:fadingMode="fade_out" />
   
       <changeBounds
               android:duration="5000"
               android:interpolator="@android:interpolator/accelerate_decelerate" />
   
       <fade
               android:duration="1000"
               android:fadingMode="fade_in" />
   </transitionSet>
   ```

   

4. 使用

   `activity_layout.xml`

   ```xml
   <?xml version="1.0" encoding="utf-8"?>
   <androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
           xmlns:app="http://schemas.android.com/apk/res-auto"
           xmlns:tools="http://schemas.android.com/tools"
           android:layout_width="match_parent"
           android:layout_height="match_parent">
   
       <FrameLayout
               android:id="@+id/mContainer"
               android:layout_width="match_parent"
               android:layout_height="match_parent" />
   
       <Button
               android:id="@+id/btnTrans"
               app:layout_constraintBottom_toBottomOf="parent"
               android:layout_width="match_parent"
               android:layout_height="wrap_content"
               android:text="translation"/>
   
   </androidx.constraintlayout.widget.ConstraintLayout>
   ```

   ```kotlin
    var i = 1
   
       override fun onCreate(savedInstanceState: Bundle?) {
           super.onCreate(savedInstanceState)
           setContentView(R.layout.activity_translation_animator)
   
   
           val (transition, scene1, scene2) = initTransXML()
   
           btnTrans.setOnClickListener {
               if (i % 2 > 0)
                   transition.transitionTo(scene1)
               else transition.transitionTo(scene2)
   
               i++
           }
       }
   
       /***
        * xml 形式
        */
       private fun initTransXML(): Triple<TransitionManager, Scene, Scene> {
           val transitionInflater = TransitionInflater.from(this)
           val transition =
               transitionInflater.inflateTransitionManager(R.transition.trans_manager, mContainer)
   
           val scene1 = Scene.getSceneForLayout(mContainer, R.layout.layout_translation_before, this)
           val scene2 = Scene.getSceneForLayout(mContainer, R.layout.layout_translation_after, this)
           return Triple(transition, scene1, scene2)
       }
   
       /***
        * 代码动态形式
        */
       @RequiresApi(Build.VERSION_CODES.LOLLIPOP)
       private fun initTransCode(mScene: Scene) {
           val changeBounds = ChangeBounds().apply { duration = 2000; }
           val fadeIn = Fade(Fade.MODE_IN).apply { duration = 2000; }
           val fadeOut = Fade(Fade.MODE_OUT).apply { duration = 2000; }
           val transitionSet = TransitionSet().apply { ordering = TransitionSet.ORDERING_SEQUENTIAL }
           transitionSet.addTransition(changeBounds)
               .addTransition(fadeIn)
               .addTransition(fadeOut)
           TransitionManager.go(mScene, transitionSet)
       }
   ```

   