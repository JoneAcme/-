# Java注解

[TOC]

## 注解的定义  
通过@interface 关键字定义  


```
public  @interface  TestAnnotation {
}
```
## 元注解  
元注解：  
    可以注解到注解上的注解，是一种基本注解

- @Retention  
- @Documented  
- @Target  
- @Inherited  
- @Repeatable

### @Retention 
Retention 英文保留的意思  
当@Retention 应用到一个注解伤的时候，解释说明了这个注解的的存活时间。它的取值范围和含义如下：
| 取值                    | 含义                                                         |
| ----------------------- | ------------------------------------------------------------ |
| RetentionPolicy.SOURCE  | 注解只在源码阶段保留，在编译器进行编译时它将被丢弃忽视。     |
| RetentionPolicy.CLASS   | 注解只被保留到编译进行的时候，它并不会被加载到 JVM 中。      |
| RetentionPolicy.RUNTIME | 注解可以保留到程序运行的时候，它会被加载进入到 JVM 中，所以在程序运行时可以获取到它们。 |
例：

```
@Retention(RetentionPolicy.RUNTIME)
public @interface TestAnnotation {
}
```
### @Documented 
作用是能够将注解中的元素包含到javaDoc中  
### @Target  
Target 目标的意思，制定了注解运用的地方。也就是说当一个注解被@Target 注解时，这个注解就被限定了运用的场景。比如，只能注解方法、类、方法参数等。具体取值如下：  

| 取值                        | 说明                                       |
| --------------------------- | ------------------------------------------ |
| ElementType.ANNOTATION_TYPE | 可以给一个注解进行注解                     |
| ElementType.CONSTRUCTOR     | 可以给构造方法进行注解                     |
| ElementType.FIELD           | 可以给属性进行注解                         |
| ElementType.LOCAL_VARIABLE  | 可以给局部变量进行注解                     |
| ElementType.METHOD          | 可以给方法进行注解                         |
| ElementType.PACKAGE         | 可以给一个包进行注解                       |
| ElementType.PARAMETER       | 可以给一个方法内的参数进行注解             |
| ElementType.TYPE            | 可以给一个类型进行注解，比如类、接口、枚举 |


### @Inherited
Inherited 是集成的意思，并不是说注解本身刻意继承，而是说如果一个超类被@Inherited注解过的注解进行注解的话，如果它的自雷没有被任何注解应用，那么这个子类就继承了超类的注解。

```
@Inherited
@Retention(RetentionPolicy.RUNTIME)
@interface Test {}


@Test
public class A {}


public class B extends A {}
```
说明：注解Test被@Inherited修饰后，A被Test注解，B继承A，并且B没有被任何注解所注解，则B也拥有Test这个注解。

### @Repeatable
Repeatable 可重复的意。 此注解是java 1.8才加进来。  
什么注解才会被多次引用呢，通常是注解的值刻意同事取多个。  

```
@interface Persons {
    Person[]  value();
}


@Repeatable(Persons.class)
@interface Person{
    String role default "";
}


@Person(role="artist")
@Person(role="coder")
@Person(role="PM")
public class SuperMan{

}


```
说明：可以理解为一个人有多重身份。  
@Repeatable 后面括号中的类，相当于一个容器注解。
## 容器注解  
本身也是一个注解，是用来存放其他注解。

```
@interface Persons {
    Person[]  value();
}
```
按照规定，容器注解中必须要有一个value属性， 属性类型是一个被@Repeatable注解过的注解数组。

## 注解的属性  
注解的属性也加成员变量。注解只有成员变量，没有方法。  
注解的成员变量在注解定义中以无形参的方法形式生命，其方法定义了该成员变量的名字，其返回值定义了该成员变量的额类型。

### 多个属性
```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface TestAnnotation {

    int id();

    String msg();

}
```
赋值的方式是在注解的括号内以 value=”” 形式，多个属性之前用","隔开：

```
@TestAnnotation(id=3,msg="hello annotation") 
public class Test {

}
```
注意：
    注解中定义属性时，他的类型必须是巴中几倍类型外加类、接口、注解及它们的数组。  
注解属性可以有默认值，需要用default 关键字指定：  

```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface TestAnnotation {

    public int id() default -1;

    public String msg() default "Hi";

}

```
### 单独属性
如果一个注解内只有一个名字为value 的属性时，应用这个注解时可以直接将属性值填写在括号内。

```
public @interface Check {
    String value();
}
```

```
@Check("hi")
int a;

// or

@Check(value="hi")
int a;
```
### 无属性

```
public @interface Perform {}
```
如果注解无属性，那么应用时可以连括号都省略

```
@Perform
public void testMethod(){}
```
## java 预置注解
### @Deprecated
用来标记过时的元素
### @Override
复写父类方法的修饰
### @SuppresssWarnings
阻止警告
### @SafeVarargs
参数安全类注解。目的是提醒开发者不要用参数做一些不安全的操作
### @FunctionInterface
函数式接口注解，java 1.8引入。  
函数式接口可以很容易转换文Lamabda表达式，待具体查询
```
@FunctionalInterface
public interface Runnable {
    /**
     * When an object implementing interface <code>Runnable</code> is used
     * to create a thread, starting the thread causes the object's
     * <code>run</code> method to be called in that separately executing
     * thread.
     * <p>
     * The general contract of the method <code>run</code> is that it may
     * take any action whatsoever.
     *
     * @see     java.lang.Thread#run()
     */
    public abstract void run();
}

```
## 注解的提取
### 注解与反射
注解是通过反射获取。首先可以通过Class对象的isAnnotationPresnt() 判断是否应用了某个注解：

```
public boolean isAnnotationPresent(Class<? extends Annotation> annotationClass) {}
```
然后通过 getAnnotation() 方法来获取 Annotation 对象。  

```
 public <A extends Annotation> A getAnnotation(Class<A> annotationClass) {}
```
或者是 getAnnotations() 方法。

```
public Annotation[] getAnnotations() {}
```
前一种方法返回指定类型的注解，后一种方法返回注解到这个元素上的所有注解。

```
@TestAnnotation()
public class Test {

    public static void main(String[] args) {

        boolean hasAnnotation = Test.class.isAnnotationPresent(TestAnnotation.class);

        if ( hasAnnotation ) {
            TestAnnotation testAnnotation = Test.class.getAnnotation(TestAnnotation.class);

            System.out.println("id:"+testAnnotation.id());
            System.out.println("msg:"+testAnnotation.msg());
        }

    }

}

```
**注意**：：  
如果一个注解要在运行时被成功提取，那么 @Retention(RetentionPolicy.RUNTIME) 是必须的。  

```
@TestAnnotation(msg="hello")
public class Test {

    @Check(value="hi")
    int a;


    @Perform
    public void testMethod(){}


    @SuppressWarnings("deprecation")
    public void test1(){
        Hero hero = new Hero();
        hero.say();
        hero.speak();
    }


    public static void main(String[] args) {

        boolean hasAnnotation = Test.class.isAnnotationPresent(TestAnnotation.class);

        if ( hasAnnotation ) {
            TestAnnotation testAnnotation = Test.class.getAnnotation(TestAnnotation.class);
            //获取类的注解
            System.out.println("id:"+testAnnotation.id());
            System.out.println("msg:"+testAnnotation.msg());
        }


        try {
            Field a = Test.class.getDeclaredField("a");
            a.setAccessible(true);
            //获取一个成员变量上的注解
            Check check = a.getAnnotation(Check.class);

            if ( check != null ) {
                System.out.println("check value:"+check.value());
            }

            Method testMethod = Test.class.getDeclaredMethod("testMethod");

            if ( testMethod != null ) {
                // 获取方法中的注解
                Annotation[] ans = testMethod.getAnnotations();
                for( int i = 0;i < ans.length;i++) {
                    System.out.println("method testMethod annotation:"+ans[i].annotationType().getSimpleName());
                }
            }
        } catch (NoSuchFieldException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
            System.out.println(e.getMessage());
        } catch (SecurityException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
            System.out.println(e.getMessage());
        } catch (NoSuchMethodException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
            System.out.println(e.getMessage());
        }



    }

}


```
## 注解使用
> 注解是一系列元数据，它提供数据用来解释程序代码，但是注解并非是所解释的代码本身的一部分。注解对于代码的运行效果没有直接影响。  

  注解有许多用处，主要如下： 
  - 提供信息给编译器： 编译器可以利用注解来探测错误和警告信息 
  - 编译阶段时的处理：  软件工具可以用来利用注解信息来生成代码、Html文档或者做其它相应处理。 
  - 运行时的处理：  某些注解可以在程序运行的时候接受代码的提取

注解主要针对的是编译器和其它工具软件(SoftWare tool)。

当开发者使用了Annotation 修饰了类、方法、Field 等成员之后，这些 Annotation 不会自己生效，必须由开发者提供相应的代码来提取并处理 Annotation 信息。这些处理提取和处理 Annotation 的代码统称为 APT（Annotation Processing Tool)。

应用举例：
1. 测试  

    —— 程序员 A : 我写了一个类，它的名字叫做 NoBug，因为它所有的方法都没有错误。   
    —— 我：自信是好事，不过为了防止意外，让我测试一下如何？   
    —— 程序员 A: 怎么测试？   
    —— 我：把你写的代码的方法都加上 @Jiecha 这个注解就好了。   
    —— 程序员 A: 好的。  
    NoBug.java  
    
    ```
    package ceshi;
    import ceshi.Jiecha;
    
    
    public class NoBug {
    
        @Jiecha
        public void suanShu(){
            System.out.println("1234567890");
        }
        @Jiecha
        public void jiafa(){
            System.out.println("1+1="+1+1);
        }
        @Jiecha
        public void jiefa(){
            System.out.println("1-1="+(1-1));
        }
        @Jiecha
        public void chengfa(){
            System.out.println("3 x 5="+ 3*5);
        }
        @Jiecha
        public void chufa(){
            System.out.println("6 / 0="+ 6 / 0);
        }
    
        public void ziwojieshao(){
            System.out.println("我写的程序没有 bug!");
        }
    
    }
    
    ```
    自定义注解：
    
    ```
    package ceshi;
    
    import java.lang.annotation.Retention;
    import java.lang.annotation.RetentionPolicy;
    
    @Retention(RetentionPolicy.RUNTIME)
    public @interface Jiecha {
    
    }
    
    ```
    测试相应方法：
    
    ```
    package ceshi;
    
    import java.lang.reflect.InvocationTargetException;
    import java.lang.reflect.Method;
    
    
    
    public class TestTool {
    
        public static void main(String[] args) {
            // TODO Auto-generated method stub
    
            NoBug testobj = new NoBug();
    
            Class clazz = testobj.getClass();
    
            Method[] method = clazz.getDeclaredMethods();
            //用来记录测试产生的 log 信息
            StringBuilder log = new StringBuilder();
            // 记录异常的次数
            int errornum = 0;
    
            for ( Method m: method ) {
                // 只有被 @Jiecha 标注过的方法才进行测试
                if ( m.isAnnotationPresent( Jiecha.class )) {
                    try {
                        m.setAccessible(true);
                        m.invoke(testobj, null);
    
                    } catch (Exception e) {
                        // TODO Auto-generated catch block
                        //e.printStackTrace();
                        errornum++;
                        log.append(m.getName());
                        log.append(" ");
                        log.append("has error:");
                        log.append("\n\r  caused by ");
                        //记录测试过程中，发生的异常的名称
                        log.append(e.getCause().getClass().getSimpleName());
                        log.append("\n\r");
                        //记录测试过程中，发生的异常的具体信息
                        log.append(e.getCause().getMessage());
                        log.append("\n\r");
                    } 
                }
            }
    
    
            log.append(clazz.getSimpleName());
            log.append(" has  ");
            log.append(errornum);
            log.append(" error.");
    
            // 生成测试报告
            System.out.println(log.toString());
    
        }
    
    }
    
    ```
    结果：
    
    ```
    1234567890
    1+1=11
    1-1=0
    3 x 5=15
    chufa has error:
    
      caused by ArithmeticException
    
    / by zero
    
    NoBug has  1 error.
    ```
2. 运用地方：JUnit、ButterKnife、Dagger2、Retrofit

## 总结  
1. 如果注解难于理解，你就把它类同于标签，标签为了解释事物，注解为了解释代码。  
2. 注解的基本语法，创建如同接口，但是多了个 @ 符号。  
3. 注解的元注解。
4. 注解的属性。
5. 注解主要给编译器及工具类型的软件用的。  
6. 注解的提取需要借助于 Java 的反射技术，反射比较慢，所以注解使用时也需要谨慎计较时间成本。