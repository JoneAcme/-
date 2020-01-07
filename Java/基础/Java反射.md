# Java反射

### 概念

Java 反射机制在程序**运行时**，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性。这种 **动态的获取信息**以及 **动态调用对象的方法**的功能称为 **java 的反射机制**。

反射机制很重要的一点就是“运行时”，其使得我们可以在程序运行时加载、探索以及使用编译期间完全未知的 `.class`文件。换句话说，Java 程序可以加载一个运行时才得知名称的 `.class`文件，然后获悉其完整构造，并生成其对象实体、或对其 fields（变量）设值、或调用其 methods（方法）。

### 实例

为使得测试结果更加明显，我首先定义了一个 `FatherClass`类（默认继承自 `Object`类），然后定义一个继承自 `FatherClass`类的 `SonClass`类，如下所示。可以看到测试类中变量以及方法的访问权限不是很规范，是为了更明显得查看测试结果而故意设置的，实际项目中不提倡这么写。

```java
public class FatherClass {
    public String mFatherName;
    public int mFatherAge;

    public void printFatherMsg(){}
}
```

```java
public class SonClass extends FatherClass{

    private String mSonName;
    protected int mSonAge;
    public String mSonBirthday;

    public void printSonMsg(){
        System.out.println("Son Msg - name : "
                + mSonName + "; age : " + mSonAge);
    }

    private void setSonName(String name){
        mSonName = name;
    }

    private void setSonAge(int age){
        mSonAge = age;
    }

    private int getSonAge(){
        return mSonAge;
    }

    private String getSonName(){
        return mSonName;
    }
}
```

#### 1. 获取类的所有变量信息

```java
/**
 * 通过反射获取类的所有变量
 */
private static void printFields(){
    //1.获取并输出类的名称
    Class mClass = SonClass.class;
    System.out.println("类的名称：" + mClass.getName());

    //2.1 获取所有 public 访问权限的变量
    // 包括本类声明的和从父类继承的
    Field[] fields = mClass.getFields();

    //2.2 获取所有本类声明的变量（不问访问权限）
    //Field[] fields = mClass.getDeclaredFields();

    //3. 遍历变量并输出变量信息
    for (Field field :
            fields) {
        //获取访问权限并输出
        int modifiers = field.getModifiers();
        System.out.print(Modifier.toString(modifiers) + " ");
        //输出变量的类型及变量名
        System.out.println(field.getType().getName()
                 + " " + field.getName());
    }
}
```

输出结果：

- `getFields()`  public 访问权限的变量

```
  类的名称：obj.SonClass
  public java.lang.String mSonBirthday
  public java.lang.String mFatherName
  public int mFatherAge
```

- `getDeclaredFields()` ， 所有成员变量，不问访问权限。

```java
  类的名称：obj.SonClass
  private java.lang.String mSonName
  protected int mSonAge
  public java.lang.String mSonBirthday
```

#### 2.获取类的所有方法信息

```java
/**
 * 通过反射获取类的所有方法
 */
private static void printMethods(){
    //1.获取并输出类的名称
    Class mClass = SonClass.class;
    System.out.println("类的名称：" + mClass.getName());

    //2.1 获取所有 public 访问权限的方法
    //包括自己声明和从父类继承的
    Method[] mMethods = mClass.getMethods();

    //2.2 获取所有本类的的方法（不问访问权限）
    //Method[] mMethods = mClass.getDeclaredMethods();

    //3.遍历所有方法
    for (Method method :
            mMethods) {
        //获取并输出方法的访问权限（Modifiers：修饰符）
        int modifiers = method.getModifiers();
        System.out.print(Modifier.toString(modifiers) + " ");
        //获取并输出方法的返回值类型
        Class returnType = method.getReturnType();
        System.out.print(returnType.getName() + " "
                + method.getName() + "( ");
        //获取并输出方法的所有参数
        Parameter[] parameters = method.getParameters();
        for (Parameter parameter:
             parameters) {
            System.out.print(parameter.getType().getName()
                    + " " + parameter.getName() + ",");
        }
        //获取并输出方法抛出的异常
        Class[] exceptionTypes = method.getExceptionTypes();
        if (exceptionTypes.length == 0){
            System.out.println(" )");
        }
        else {
            for (Class c : exceptionTypes) {
                System.out.println(" ) throws "
                        + c.getName());
            }
        }
    }
}
```

-  `getMethods()` 

  ```
    类的名称：obj.SonClass
    public void printSonMsg(  )
    public void printFatherMsg(  )
    public final void wait(  ) throws java.lang.InterruptedException
    public final void wait( long arg0,int arg1, ) throws java.lang.InterruptedException
    public final native void wait( long arg0, ) throws java.lang.InterruptedException
    public boolean equals( java.lang.Object arg0, )
    public java.lang.String toString(  )
    public native int hashCode(  )
    public final native java.lang.Class getClass(  )
    public final native void notify(  )
    public final native void notifyAll(  )
  ```

- `getDeclaredMethods()`

  ```
    类的名称：obj.SonClass
    private int getSonAge(  )
    private void setSonAge( int arg0, )
    public void printSonMsg(  )
    private void setSonName( java.lang.String arg0, )
    private java.lang.String getSonName(  )
  ```


### 访问或操作类的私有变量和方法

```java
public class TestClass {

    private String MSG = "Original";

    private void privateMethod(String head , int tail){
        System.out.print(head + tail);
    }

    public String getMsg(){
        return MSG;
    }
}
```

#### 1.访问私有方法

```java
/**
 * 访问对象的私有方法
 * 为简洁代码，在方法上抛出总的异常，实际开发别这样
 */
private static void getPrivateMethod() throws Exception{
    //1. 获取 Class 类实例
    TestClass testClass = new TestClass();
    Class mClass = testClass.getClass();

    //2. 获取私有方法
    //第一个参数为要获取的私有方法的名称
    //第二个为要获取方法的参数的类型，参数为 Class...，没有参数就是null
    //方法参数也可这么写 ：new Class[]{String.class , int.class}
    Method privateMethod =
            mClass.getDeclaredMethod("privateMethod", String.class, int.class);

    //3. 开始操作方法
    if (privateMethod != null) {
        //获取私有方法的访问权
        //只是获取访问权，并不是修改实际权限
        privateMethod.setAccessible(true);

        //使用 invoke 反射调用私有方法
        //privateMethod 是获取到的私有方法
        //testClass 要操作的对象
        //后面两个参数传实参
        privateMethod.invoke(testClass, "Java Reflect ", 666);
    }
}
```

注意：第3步中的 `setAccessible(true)` 方法，是获取私有方法的访问权限，如果不加会报异常 **IllegalAccessException**，因为当前方法访问权限是“private”的，如下：

```
java.lang.IllegalAccessException: Class MainClass can not access a member of class obj.TestClass with modifiers "private"
```

正常运行结果：

```
Java Reflect 666
```

#### 2.修改私有变量

```java
/**
 * 修改对象私有变量的值
 * 为简洁代码，在方法上抛出总的异常
 */
private static void modifyPrivateFiled() throws Exception {
    //1. 获取 Class 类实例
    TestClass testClass = new TestClass();
    Class mClass = testClass.getClass();

    //2. 获取私有变量
    Field privateField = mClass.getDeclaredField("MSG");

    //3. 操作私有变量
    if (privateField != null) {
        //获取私有变量的访问权
        privateField.setAccessible(true);

        //修改私有变量，并输出以测试
        System.out.println("Before Modify：MSG = " + testClass.getMsg());

        //调用 set(object , value) 修改变量的值
        //privateField 是获取到的私有变量
        //testClass 要操作的对象
        //"Modified" 为要修改成的值
        privateField.set(testClass, "Modified");
        System.out.println("After Modify：MSG = " + testClass.getMsg());
    }
}
```

结果：

```
Before Modify：MSG = Original
After Modify：MSG = Modified
```

#### 3.修改私有常量

常量是指使用 `final` 修饰符修饰的成员属性，与变量的区别就在于有无 `final` 关键字修饰。

Java 虚拟机（JVM）在编译 `.java` 文件得到 `.class` 文件时，会优化我们的代码以提升效率。其中一个优化就是：JVM 在编译阶段会把引用常量的代码替换成具体的常量值，如下所示（部分代码）。

编译前的 `.java` 文件：

```java
//注意是 String  类型的值
private final String FINAL_VALUE = "hello";

if(FINAL_VALUE.equals("world")){
    //do something
}
```

编译后得到的 `.class` 文件（当然，编译后是没有注释的）：

```java
private final String FINAL_VALUE = "hello";
//替换为"hello"
if("hello".equals("world")){
    //do something
}
```

并不是所有常量都会优化。经测试对于 `int` 、`long` 、`boolean` 以及 `String` 这些基本类型 JVM 会优化，而对于 `Integer` 、`Long` 、`Boolean` 这种包装类型，或者其他诸如 `Date`、`Object` 类型则不会被优化。

**对于基本类型的静态常量，JVM 在编译阶段会把引用此常量的代码替换成具体的常量值**。



**我们在程序运行时刻依然可以使用反射修改常量的值（后面会代码验证），但是 JVM 在编译阶段得到的 .class 文件已经将常量优化为具体的值，在运行阶段就直接使用具体的值了，所以即使修改了常量的值也已经毫无意义了**。

验证：

```java
//String 会被 JVM 优化
private final String FINAL_VALUE = "FINAL";

public String getFinalValue(){
    //剧透，会被优化为: return "FINAL" ,拭目以待吧
    return FINAL_VALUE;
}
```

接下来，是修改常量的值：

```java
/**
 * 修改对象私有常量的值
 * 为简洁代码，在方法上抛出总的异常，实际开发别这样
 */
private static void modifyFinalFiled() throws Exception {
    //1. 获取 Class 类实例
    TestClass testClass = new TestClass();
    Class mClass = testClass.getClass();

    //2. 获取私有常量
    Field finalField = mClass.getDeclaredField("FINAL_VALUE");

    //3. 修改常量的值
    if (finalField != null) {

        //获取私有常量的访问权
        finalField.setAccessible(true);

        //调用 finalField 的 getter 方法
        //输出 FINAL_VALUE 修改前的值
        System.out.println("Before Modify：FINAL_VALUE = "
                + finalField.get(testClass));

        //修改私有常量
        finalField.set(testClass, "Modified");

        //调用 finalField 的 getter 方法
        //输出 FINAL_VALUE 修改后的值
        System.out.println("After Modify：FINAL_VALUE = "
                + finalField.get(testClass));

        //使用对象调用类的 getter 方法
        //获取值并输出
        System.out.println("Actually ：FINAL_VALUE = "
                + testClass.getFinalValue());
    }
}
```

结果：

```
Before Modify：FINAL_VALUE = FINAL
After Modify：FINAL_VALUE = Modified
Actually ：FINAL_VALUE = FINAL
```

第一句打印修改前 `FINAL_VALUE`的值，没有异议；

第二句打印修改后常量的值，说明`FINAL_VALUE`确实通过反射修改了；

第三句打印通过 `getFinalValue()`方法获取的 `FINAL_VALUE`的值，但还是初始值，导致修改无效！

![TestClass.class 文件](https://upload-images.jianshu.io/upload_images/6760361-ad58d14f098ea634?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



以上截图是jvm编译后得到的Class文件，`getFinalValue()` 方法直接 `return "FINAL"`！同时也说明了，**程序运行时是根据编译后的 .class 来执行的**。



1. 构造函数

   Java 允许我们声明常量时不赋值，但必须在构造函数中赋值。你可能会问我为什么要说这个，这就解释：

   我们修改一下 `TestClass` 类，在声明常量时不赋值，然后添加构造函数并为其赋值，大概看一下修改后的代码（部分代码 ）：

   ```java
   public class TestClass {
   
       //......
       private final String FINAL_VALUE;
   
       //构造函数内为常量赋值 
       public TestClass(){
           this.FINAL_VALUE = "FINAL";
       }
       //......
   }
   ```

   结果：

   ```
   Before Modify：FINAL_VALUE = FINAL
   After Modify：FINAL_VALUE = Modified
   Actually ：FINAL_VALUE = Modified
   ```

   .Class文件

   ![TestClass.class 文件](https://upload-images.jianshu.io/upload_images/6760361-7e3709007945690d?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

   解释一下：我们将赋值放在构造函数中，构造函数是我们运行时 new 对象才会调用的，所以就不会像之前直接为常量赋值那样，在编译阶段将 `getFinalValue()`方法优化为返回常量值，而是指向 `FINAL_VALUE`，这样我们在运行阶段通过反射修改敞亮的值就有意义啦。但是，看得出来，程序还是有优化的，将构造函数中的赋值语句优化了。

    **程序运行时是根据编译后的 .class 来执行的**

2. 三目运算

   不使用构造函数，也可以成功修改常量的值，但原理上都一样。去掉构造函数，将声明常量的语句改为使用三目表达式赋值：

   ```java
   private final String FINAL_VALUE
           = null == null ? "FINAL" : null;
   ```

   其实，上述代码等价于直接为 `FINAL_VALUE` 赋值 "FINAL"，但是他就是可以！至于为什么，你这么想：`null == null ? "FINAL" : null` 是在运行时刻计算的，在编译时刻不会计算，也就不会被优化，所以你懂得。

3. 总结：总结来说，不管使用构造函数还是三目表达式，根本上都是**避免在编译时刻被优化**，这样我们通过反射修改常量之后才有意义！

   **最后的强调**：

   	无论**直接为常量赋值**、 **通过构造函数为常量赋值**还是 **使用三目运算符**，实际上我们都能通过反射成功修改常量的值。而上面说的修改"成功"与否是指：**我们在程序运行阶段通过反射肯定能修改常量值，但是实际执行优化后的 .class 文件时，修改的后值真的起到作用了吗？换句话说，就是编译时是否将常量替换为具体的值了？如果替换了，再怎么修改常量的值都不会影响最终的结果了，不是吗？**