# UI优化

[TOC]

## 布局优化

### 1、 减少布局的层级（嵌套）

- 原理：布局层级少 ->> 绘制的工作量少 ->> 绘制速度快 ->> 性能提高
- 优化方式：使用布局标签`<merge>`& 合适选择布局类型

#### 1.1 使用布局标签\<merge>

- 作用
  减少 布局层级

> 配合`<include>`标签使用，可优化 加载布局文件时的资源消耗

- 具体使用

```xml
  // 使用说明：
    // 1. <merge>作为被引用布局A的根标签
    // 2. 当其他布局通过<include>标签引用布局A时，布局A中的<merge>标签内容（根节点）会被去掉，在<include>里存放的是布局A中的<merge>标签内容（根节点）的子标签（即子节点），以此减少布局文件的层次

   /** 
     * 实例说明：在上述例子，在布局B中 通过<include>标签引用布局C
     * 此时：布局层级为 =  RelativeLayout ->> Button 
     *                                  —>> RelativeLayout ->> Button
     *                                                     ->> TextView
     * 现在使用<merge>优化：将 被引用布局C根标签 的RelativeLayout 改为 <merge>
     * 在引用布局C时，布局C中的<merge>标签内容（根节点）会被去掉，在<include>里存放的是布局C中的<merge>标签内容（根节点）的子标签（即子节点）
     * 即 <include>里存放的是：<Button>、<TextView>
     * 此时布局层级为 =  RelativeLayout ->> Button 
     *                                ->> Button
     *                                ->> TextView
     * 即 已去掉之前无意义、多余的<RelativeLayout>
     */  

     // 被引用的公共部分：布局C = layout_c.xml
     <?xml version="1.0" encoding="utf-8"?>
    <merge xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="match_parent" >
     
        <Button
            android:id="@+id/button"
            android:layout_width="match_parent"
            android:layout_height="@dimen/dp_10"/>

        <TextView
            android:id="@+id/textview"
            android:layout_width="match_parent"
            android:layout_height="@dimen/dp_10"/>
     
    </merge>

    // 布局B：layout_b.xml
    <?xml version="1.0" encoding="utf-8"?>
    <RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="match_parent" >
     
        <Button
            android:id="@+id/Button"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:layout_marginBottom="@dimen/dp_10" />

        <include layout="@layout/layout_c.xml" />

    </RelativeLayout>
```

#### 1.2 合适选择布局类型

- 通过合理选择布局类型，从而减少嵌套
- 即：完成 复杂的`UI`效果时，**尽可能选择1个功能复杂的布局（如RelativeLayout）完成，而不要选择多个功能简单的布局（如LinerLayout）通过嵌套完成**

### 2、 提高 布局 的复用性

- 原因
  提取布局间的公共部分，通过提高布局的复用性从而减少测量 & 绘制时间
- 优化方案
  使用 布局标签 `<include>`

#### 2.1 使用 布局标签\<include>

- 作用
  实现 **布局模块化**，即 提取布局中的公共部分 供其他布局共用
- 具体使用

```xml
// 使用说明：
  // a. 通过<include>标签引入抽取的公共部分布局C
  // b. <include>标签所需属性 = 公共部分的layout属性，作用 = 指定需引入、包含的布局文件

// 实例说明：抽取 布局A、B中的公共部分布局C & 放入到布局B中使用

   /** 
     * 布局B：layout_b.xml
     */  
    <?xml version="1.0" encoding="utf-8"?>
    <RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="match_parent" >
     
        <Button
            android:id="@+id/Button"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:layout_marginBottom="@dimen/dp_10" />
        
        // 通过<include>标签引入抽取的公共部分布局C
        // <include>标签所需属性 = 公共部分的layout属性，作用 = 指定需引入、包含的布局文件
        <include layout="@layout/layout_c.xml" />
     
    </RelativeLayout>

    /** 
     * 公共部分的布局C：layout_c.xml
     */
     <?xml version="1.0" encoding="utf-8"?>
    <RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="match_parent" >
     
        <Button
            android:id="@+id/button"
            android:layout_width="match_parent"
            android:layout_height="@dimen/dp_10"/>

        <TextView
        android:id="@+id/textview"
        android:layout_width="match_parent"
        android:layout_height="@dimen/dp_10"/>
     
    </RelativeLayout>
```

### 3、 减少初次测量 & 绘制时间

主要优化方案：使用 布局标签`<ViewStub>`& 尽可能少用布局属性 `wrap_content`

#### 3.1 使用 布局标签\<ViewStub>

- 作用
  **按需加载**外部引入的布局

> 注：属 轻量级`View`、不占用显示 & 位置

- 应用场景
  引入 只在特殊情况下才显示的布局（即 默认不显示）

> 如：进度显示布局、信息出错出现的提示布局等

- 具体使用

```xml
  // 使用说明：
    // 1. 先设置好预显示的布局
    // 2. 在其他布局通过<ViewStub>标签引入外部布局（类似<include>）；注：此时该布局还未被加载显示
    // 3. 只有当ViewStub被设置为可见 / 调用了ViewStub.inflate()时，ViewStub所指向的布局文件才会被inflate 、实例化，最终 显示<ViewStub>指向的布局

   /** 
     * 实例说明：在布局A中引入布局B，只有在特定时刻C中才显示
     */  

     // 步骤1：先设置好预显示的布局B = layout_b.xml
     <?xml version="1.0" encoding="utf-8"?>
     <RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="match_parent" >
     
        <Button
            android:id="@+id/button"
            android:layout_width="match_parent"
            android:layout_height="@dimen/dp_10"/>

        <TextView
            android:id="@+id/textview"
            android:layout_width="match_parent"
            android:layout_height="@dimen/dp_10"/>
     
    </RelativeLayout>

    // 步骤2：在布局A通过<ViewStub>标签引入布局B（类似<include>）；注：此时该布局还未被加载显示
    // 布局A：layout_a.xml
    <?xml version="1.0" encoding="utf-8"?>
    <RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="match_parent" >
     
        <Button
            android:id="@+id/Button"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:layout_marginBottom="@dimen/dp_10" />

        <ViewStub
            android:id="@+id/Blayout"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:layout="@layout/layout_b" />

    </RelativeLayout>

    // 步骤3：只有当ViewStub被设置为可见 / 调用了ViewStub.inflate()时，ViewStub所指向的布局文件才会被inflate 、实例化，最终 显示<ViewStub>指向的布局
    ViewStub stub = (ViewStub) findViewById(R.id.Blayout);   
    stub.inflate();

// 特别注意
 // 1. ViewStub中的layout布局不能使用merge标签，否则会报错
 // 2. ViewStub的inflate只能执行一次，显示了之后，就不能再使用ViewStub控制它了
// 3. 与View.setVisible(View.Gone)的区别：View 的可见性设置为 gone 后，在inflate 时，该View 及其子View依然会被解析；而使用ViewStub就能避免解析其中指定的布局文件，从而节省布局文件的解析时间 & 内存的占用
```

#### 3.2 尽可能少用布局属性 wrap_content

布局属性 `wrap_content`会增加布局测量时计算成本，应尽可能少用

> 在已知宽高为固定值时，不使用`wrap_content`

## 绘制优化

### 1、 降低View.onDraw() 的复杂度

#### 1.1 onDraw()中不要创建局部对象

#### 1.2 避免onDraw()中执行大量操作&耗时操作

### 2、 避免过度绘制

#### 2.1 移除默认的window背景

#### 2.2 移除控件中不必要的背景

#### 2.3 减少布局文件层级

#### 2.4 自定义View优化：使用clipRect()、quickReject()

clipRect() :给 Canvas 设置一个裁剪区域，只有在该区域内才会被绘制，区域之外的都不绘制

quickReject(): 判断和某个矩形相交，如果相交，则可跳过相交的区域，从而减少过度绘制

### 3、 其他优化

#### 3.1 OpenGL 提高绘制性能

#### 3.2 尽量为所有分辨率创建资源，减少不必要的缩放

#### 3.3 Render Javascript 

## 布局调优工具

- 背景
  尽管已经注意到上述的优化策略，但实际开发中难免还是会出现布局性能的问题
- 解决方案
  使用 布局调优工具

> 此处主要介绍 常用的：`hierarchy viewer`、`Lint`、`Systrace`

### 1、 Hierarchy Viewer

- 简介
  `Android Studio`提供的UI性能检测工具。
- 作用
  可视化获得UI布局设计结构 & 各种属性信息，帮助我们优化布局设计

> 即 ：方便查看`Activity`布局，各个`View`的属性、布局测量-布局-绘制的时间

- 具体使用
  [Hierarchy Viewer 使用指南](https://link.jianshu.com/?t=http%3A%2F%2Fdeveloper.android.com%2Ftools%2Fdebugging%2Fdebugging-ui.html)

### 2、 Lint

- 简介
  `Android Studio`提供的 代码扫描分析工具
- 作用
  扫描、发现代码结构 / 质量问题；提供解决方案

> 1. 该过程不需手写测试用例
> 2. `Lint`发现的每个问题都有描述信息 & 等级（和测试发现 bug 很相似），可方便定位问题 & 按照严重程度进行解决

- 具体使用
  [Lint 使用指南 ](https://link.jianshu.com/?t=http%3A%2F%2Fblog.csdn.net%2Fu011240877%2Farticle%2Fdetails%2F54141714)

### 3、 Systrace

- 简介
  `Android 4.1`以上版本提供的性能数据采样 & 分析工具
- 作用
  检测 `Android`系统各个组件随着时间的运行状态 & 提供解决方案

> 1. 收集 等运行信息，从而帮助开发者更直观地分析系统瓶颈，改进性能
> 检测范围包括：`Android`关键子系统（如`WindowManagerService`等 `Framework`部分关键模块）、服务、View系统
> 2. 功能包括：跟踪系统的`I/O`操作、内核工作队列、`CPU`负载等，在 UI 显示性能分析上提供很好的数据，特别是在动画播放不流畅、渲染卡等问题上

- 具体使用
  [Systrace 使用指南 ](https://link.jianshu.com/?t=http%3A%2F%2Fgityuan.com%2F2016%2F01%2F17%2Fsystrace%2F)

## 