# 读书笔记：Activity--LaunchMode



## 四种启动模式简介：

1. standard ： 默认默认，每次启动Activity都会新建一个实例
2. singleTop：栈顶服用，启动时判断栈顶是否存在实例，如果存在，则复用该实例。生命周期不再调用onCreate、onStart，而是会调用onNewIntent
3. singleTask：栈内复用，如果当前栈内存在该Activity的实例，那么会移除该实例上的所有（默认具有clearTop效果）；如果栈内没有，则新建实例，压入对应栈
4. singleInstance：唯一实例，singleTask加强版，效果为singleTop+独立的栈

 **此处需要注意的是**：

每一个Activity启动时，都需要指定对应栈，默认为包名。

在standard模式下，Activity会默认进入启动它的Activity所属的任务栈中。

为什么使用Application、Service的Context启动Activity时，如果不指定FLAG_ACTIVITY_NEW_TASK，会ERROR，因为当前Context非Activity类型的Context，没有对应的任务栈。

## 使用方法

1. AndroidMenifest.xml
2. Intent.addFlags()

其中，Intent.addFlags 优先级更高。

不同点：

1. AndroidMenifest.xml 无法指定clearTop选项（FLAG_ACTIVITY_CLEAR_TOP）
2. Intent.addFlags 无法指定为SingleInstance 启动模式

## 重点参数说明

1. TaskAffinity：为Activity的启动指定对应栈，默认为包名

2. AllowTaskReparenting：跳转其他应用时，是否转移栈

   > 说明： 如 A应用跳转B应用的C界面，此时回到桌面，打开B应用
   >
   > 如果AllowTaskReparenting的值为true，那么B应用显示的为C界面而不是B应用的主页



## 例子说明

### 假设1

​	有两个栈：前台任务栈： A -- B ，后台任务栈：C -- D ，上述描述右侧为栈顶，C、D的启动模式为SingTask

​	现B启动D，现在栈中是什么情况？

​	答：A -- B -- C -- D。前台任务栈中的B启动后台任务栈的D时，整个后台任务栈都会被切换到前台栈中

### 假设2

​	现有Activity ： A、B、C，包名为com.example.tast 

​	A：launchMode = standard

​		taskAffinity未指定

​	B、C：launchMode = singleTask 

​		taskAffinity = com.example.tast1

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
