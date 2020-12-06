Android 各个版本主要变更

[toc]



## Android5.0

### 1. 虚拟机

在 **Android 5.0** 中，全面由**Dalvik**转用**ART**(Android Runtime)编译。大大提高了性能。

- 预先 (AOT) 编译
- 改进的垃圾回收 (GC)
- 改进的调试支持

### 2. Material Design 样式

​	采用全新**[Material Design](https://links.jianshu.com/go?to=http%3A%2F%2Fwiki.jikexueyuan.com%2Fproject%2Fmaterial-design%2Fmaterial-design-intro%2Fintroduction.html)**设计，页面更加的美观，立体。

### 3. 浮动通知

​	设备未锁定且其屏幕处于打开状态，通知可以显示在小型浮动窗口中，称为**浮动通知**。

### 4. 绑定服务

启动或绑定服务必须**显式启动**，如果**隐式启动**，会引发下列异常：

```csharp
 Caused by: java.lang.IllegalArgumentException: Service Intent must be explicit
```

如果非要**隐式启动**，可以使用下列方案来避免异常：

```kotlin
val intent = Intent("Service对应的Action")
intent.`package` = packageName
bindService(intent,mServiceConnection,BIND_AUTO_CREATE)
```

## Android6.0

### 1.运行时权限



### 2.低电耗模式和应用待机模式

- **低电耗模式：**如果用户拔下设备的电源插头，并在屏幕关闭后的一段时间内使其保持不活动状态，设备会进入`低电耗模式`。 在该模式下设备会尝试让`系统保持休眠状态`。在该模式下，设备会定期短时间恢复正常工作，以便进行应用同步，`还可让系统执行任何挂起的操作`。
- **应用待机模式：**应用待机模式允许系统判定应用在用户**未主动**使用它时处于**空闲状态**。当用户有一段时间未触摸应用时，系统便会作出此判定。如果拔下了设备电源插头，系统会为其视为**空闲**的应用`停用网络访问以及暂停同步和作业`。

**在低电耗模式下，您的应用会受到以下限制：**

- 暂停访问**网络**
- 系统忽略**唤醒锁定**
- 标准**AlarmManager**闹钟推迟到下一个维护期
- 系统不执行**WLAN**扫描
- 系统不允许运行**同步适配器[SyncAdapter](https://links.jianshu.com/go?to=https%3A%2F%2Fdeveloper.android.google.cn%2Ftraining%2Fsync-adapters%2Fcreating-sync-adapter)**
- 系统不允许运行**[JobScheduler](https://links.jianshu.com/go?to=https%3A%2F%2Fdeveloper.android.google.cn%2Freference%2Fandroid%2Fapp%2Fjob%2FJobScheduler.html)**

## Android7.0

### 1.文件访问权限

不允许出现以file://的形式调用隐式APP系统。

```xml
<provider
    android:name="android.support.v4.content.FileProvider"
    android:authorities="${applicationId}.fileprovider"
    android:exported="false"
    android:grantUriPermissions="true">
    <meta-data
        android:name="android.support.FILE_PROVIDER_PATHS"
        android:resource="@xml/file_path" />
</provider>
```

```xml
<paths xmlns:android="http://schemas.android.com/apk/res/android">
    <external-path name="external" path="" />
</paths>
```

### 2. 进一步优化低电耗模式

**Android 6.0**引入了`低电耗模式`，当用户设备未插接电源、处于静止状态且屏幕关闭时，该模式会推迟 **CPU和网络**活动，从而延长电池寿命。而**Android 7.0**则通过在设备未插接电源且屏幕关闭状态下、但不一定要处于`静止状态`（例如用户外出时把手持式设备装在口袋里）时应用部分**CPU和网络**限制，进一步增强了`低电耗模式`。

## Android8.0

### 1.后台执行限制

- 在后台运行的应用对**后台服务**的访问受到限制。
- 应用无法使用其清单注册大部分**隐式广播**。

### 2.后台位置限制

为节约电池电量、保持良好的用户体验和确保系统健康运行，在运行**Android 8.0**的设备上使用`后台应用时`，降低了后台应用接收`位置更新的频率`。此行为变更会影响包括**Google Play**服务在内的所有接收位置更新的应用。

### 3.通知分渠道设置

允许为要显示的每种通知类型创建用户可自定义的渠道。用户界面将通知渠道称之为`通知类别`。

### 4.[自适应图标](https://links.jianshu.com/go?to=https%3A%2F%2Fdeveloper.android.google.cn%2Fguide%2Fpractices%2Fui_guidelines%2Ficon_design_adaptive)

可以在不同设备型号上显示为不同的形状。

## Android9.0

### 1.[电源管理](https://links.jianshu.com/go?to=https%3A%2F%2Fdeveloper.android.google.cn%2Fabout%2Fversions%2Fpie%2Fpower)

- **应用待机群组：**系统将根据用户的使用模式限制应用对 CPU 或电池等设备资源的访问。
- **省电模式改进：**开启省电模式后，系统会对所有应用施加限制。 这是一项已有的功能，但在`Android 9`中得到了改进。

### 2.隐私权变更

**后台对传感器的访问受限：**`Android 9`限制后台应用访问用户输入和传感器数据的能力。 如果您的应用在运行 `Android 9`设备的后台运行，系统将对您的应用采取以下限制：

- 您的应用不能访问麦克风或摄像头。
- 使用[连续](https://links.jianshu.com/go?to=https%3A%2F%2Fsource.android.google.cn%2Fdevices%2Fsensors%2Freport-modes%23continuous)报告模式的传感器（例如加速度计和陀螺仪）不会接收事件。
- 使用[变化](https://links.jianshu.com/go?to=https%3A%2F%2Fsource.android.google.cn%2Fdevices%2Fsensors%2Freport-modes%23on-change)或[一次性](https://links.jianshu.com/go?to=https%3A%2F%2Fsource.android.google.cn%2Fdevices%2Fsensors%2Freport-modes%23one-shot)报告模式的传感器不会接收事件。

> 如果需要在运行 `Android 9`的设备上检测传感器事件，请使用[前台服务](https://links.jianshu.com/go?to=https%3A%2F%2Fdeveloper.android.google.cn%2Fguide%2Fcomponents%2Fservices.html%23Foreground)。

**限制访问通话记录：**`Android 9`引入`CALL_LOG权限组`并将`READ_CALL_LOG、WRITE_CALL_LOG和PROCESS_OUTGOING_CALLS`权限移入该组。 在之前的`Android版本`中，这些权限位于`PHONE权限组`。
 如果您的应用需要访问通话记录或者需要处理去电，则您必须向`CALL_LOG权限组`明确请求这些权限。 否则会发生`SecurityException`。

### 3.对使用非 SDK 接口的限制

为帮助确保应用**稳定性和兼容性**，此平台对某些**非SDK**`函数`和`字段`的使用进行了限制；无论您是直接访问这些`函数`和`字段`，还是通过**反射**或 **JNI** 访问，这些限制均适用。 在 `Android 9 中`，您的应用可以继续访问这些受限的接口；该平台通过 `toast` 和日志条目提醒您注意这些接口。 如果您的应用显示这样的 `toast`，则必须寻求受限接口之外的其他实现策略。

### 4.默认使用`https`

默认使用`https`，会阻止`http`请求，如果想继续使用`http`可以在清单文件中做如下配置：



```xml
<application
     ...
     android:usesCleartextTraffic="true">
...
</application>
```

### 6.前台服务

如果应用以`Android 9`或更高版本为目标平台并使用**前台服务**，则必须请求`FOREGROUND_SERVICE`权限。这是普通权限，因此，系统会自动为请求权限的应用授予此权限。
 如果以`Android 9`或更高版本为目标平台的应用尝试创建前台服务且未请求`FOREGROUND_SERVICE`，则系统会抛出`SecurityException`。

## Android10

### 1. Scoped Storage（分区存储）

android:requestLegacyExternalStorage="true"来请求使用旧的存储模式。

在下一个版本的Android中，此条配置将会失效，将强制采用外部储存限制。

以前我们习惯使用Environment.getExternalStorageDirectory()方法，那么现在可以使用getExternalFilesDir()方法（包括下载的安装包这类的文件）。如果是缓存类型文件，可以放到getExternalCacheDir()路径下。

或者使用MediaStore，将文件存至对应的媒体类型中（图片：MediaStore.Images ，视频：MediaStore.Video，音频：MediaStore.Audio），不过仅限于多媒体文件。

下面代码将图片保存到公共目录下，返回Uri：

```java
   public static Uri createImageUri(Context context) {
        ContentValues values = new ContentValues();
        // 需要指定文件信息时，非必须
        values.put(MediaStore.Images.Media.DESCRIPTION, "This is an image");
        values.put(MediaStore.Images.Media.DISPLAY_NAME, "Image.png");
        values.put(MediaStore.Images.Media.MIME_TYPE, "image/png");
        values.put(MediaStore.Images.Media.TITLE, "Image.png");
        values.put(MediaStore.Images.Media.RELATIVE_PATH, "Pictures/test");
		return context.getContentResolver().insert(MediaStore.Images.Media.EXTERNAL_CONTENT_URI, values);
    }
```

**对于媒体资源的访问：比如图片选择器这类的场景。无法直接使用File，而应使用Uri。否则报错如下：**

```
java.io.FileNotFoundException: open failed: EACCES (Permission denied)
```

比如我在适配项目中使用的图片选择器时，首先修改了Glide 通过加载File的方式显示图片。改为加载Uri的方式，否则图片无法显示出来。

Uri的获取方式还是使用MediaStore：

```java
String id = cursor.getString(cursor.getColumnIndexOrThrow(MediaStore.Images.Media._ID));

Uri uri = Uri.withAppendedPath(MediaStore.Images.Media.EXTERNAL_CONTENT_URI, id);
```

**补充**

应用在卸载后，会将App-specific目录下的数据删除，如果在AndroidManifest.xml中声明：android:hasFragileUserData="true"用户可以选择是否保留。

### 2. 权限变化

**1、在后台运行时访问设备位置信息需要权限**

Android 10 引入了 ACCESS_BACKGROUND_LOCATION 权限（危险权限）。

```xml
<uses-permission android:name="android.permission.ACCESS_BACKGROUND_LOCATION"/>
```

该权限允许应用程序在后台访问位置。如果请求此权限，则还必须请求ACCESS_FINE_LOCATION 或 ACCESS_COARSE_LOCATION权限。只请求此权限无效果。

### 2.  一些电话、蓝牙和WLAN的API需要精确位置权限

**电话:** TelephonyManager 其中的大多数方法

**WLAN：** wifi信息等

**蓝牙**

### 3. 后台启动 Activity 的限制

应用处于后台时，无法启动Activity。

比如点开一个应用会进入启动页或者广告页，一般会有几秒的延时再跳转至首页。如果这期间退到后台，将无法看到跳转过程。而在之前的版本中，会强制弹出页面至前台。

不受限的情况，主要有以下几点：

- 应用具有可见窗口，例如前台 Activity。
- 应用在前台任务的返回栈中已有的 Activity。
- 应用在 Recents 上现有任务的返回栈中已有的 Activity。Recents 就是我们的任务管理列表。
- 应用收到系统的 PendingIntent 通知。
- 应用收到它应该在其中启动界面的系统广播。示例包括 ACTION_NEW_OUTGOING_CALL 和 SECRET_CODE_ACTION。应用可在广播发送几秒钟后启动 Activity。

这一限制导致最明显的问题就是点击推送信息时，有些应用无法进行正常的跳转（具体的实现问题导致）。所以针对这类问题，可以采取PendingIntent的方式，发送通知时使用setContentIntent方法。

对于全屏 intent，注意设置最高优先级和添加USE_FULL_SCREEN_INTENT权限，这是一个普通权限。比如微信来语音或者视频通话时，弹出的接听页面就是使用这一功能。

### 4. 深色主题

### 5. 标识符和数据

**Build**

- getSerial()



**TelephonyManager**

- getImei()
- getDeviceId()
- getMeid()
- getSimSerialNumber()
- getSubscriberId()

应用必须具有 READ_PRIVILEGED_PHONE_STATE 特许权限才能正常使用以上这些方法。

### 6. 对启用和停用 WLAN 实施了限制

以 Android 10 或更高版本为目标平台的应用无法启用或停用 WLAN。WifiManager.setWifiEnabled()方法始终返回 false。

### 7. 折叠屏支持



## Android11

### 1. 分区存储

忽略Android10中的 requestLegacyExternalStorage=true配置，强制分区存储。

某些应用的核心功能可能需要访问大量的文件，例如文件管理操作、备份和恢复操作等等，此时就需要申请 **MANAGE\*EXTERNAL STORAGE** 权限。可以通过使用 **ACTION\*MANAGE\*ALL\*FILES\*ACCESS_PERMISSION** intent 操作将用户引导至一个系统设置页面，让用户为应用授予所有文件的管理权限。

### 2. 应用包可见性

**getPackageInfo(“another.app”,0)** 获取其他应用包信息时 ，会出现 **NameNotFoundException** 的异常。

对于一些特殊应用，想要获取所有包名信息：

```xml
 <uses-permission android:name="android.permission.QUERY_ALL_PACKAGES"/>
```

### 3. 权限变化

对应**摄像头、位置信息和麦克风**这几个数据类型，用户可以授予**一次性**的临时访问权限。

一次性权限的生效周期指的是：

1.应用 Activity 可见期间

2.应用转为后台后的短时间内

3.前台服务存活期间

4.当用户撤销单次授权后，应用进程退出，再次打开之后需要对应用进行重新授权期间

**位置权限**

在Android10 之前，我们通过**ACCESS\*COARRSE\*LOCATION** 或 **ACCESS\*FINE\*LOCATION(** **精确位置)** 配置即可申请前后台位置权限。

Android 11将位置权限分为前台和后台两种权限。前文说的主要是前台权限，授权方式没有变化。应用想要申请后台权限，除了需要在清单文件中额外添加 **ACCESS\*BACKGROUND\*LOCATION** 权限外，还需要应用主动引导用户到指定页面授权。

### 4. 增加应用退出原因功能

ActivityManager.getHistoricalProcessExitReasons() 方法，可以清楚地了解到应用退出的原因。