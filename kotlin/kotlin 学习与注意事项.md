## Java与Kotlin 交互的语法变化

### 1. 直接写在文件中的方法或者变量

会直接编译为public static 的方法或变量，调用如下：

```kotlin
// Utils.kt
fun test(name:String){
...
}
//main.java
public static void main(String[] args){
  UtilsKt.text("123")
}
```



### 2. 匿名内部类的调用

```kotlin
Object Test{
	fun textFun(){
	...
	}
}
//调用：
kotlin ： Test.textFun()
Java:			Test.INSTANCE.textFun();
```

Test在编译时，会被改成单例模式，构造方法改为私有，INSTANCE为其单例对象。

想要java中与kotlin中调用时写法相同，那么：

```kotlin
Object Test{
  // 添加静态注解即可
	@JvmStatic
	fun textFun(){
	...
	}
}
//调用：
kotlin ： Test.textFun()
Java:			Test.textFun();
```



### 3. Class对象

```kotlin
kotlin :	Test::Class.java
java	 :	Test.class
```

Kotlin 中，class最后编译成的是KClass，所以要获取到Java的Class，那么需要使用如上的写法



### 4. 关键字的冲突

```java
//Test.java
public class Test{
	public static String in = "in";
}
//Main.kt
fun main(...){
  var str = Test.'in'
}
```

java中使用了kotlin中的关键字作为变量名，在kotlin中使用此变量时，需用' ' 包裹

### 5. kotlin构造函数重载

使用`@JvmOverloads`注解

```kotlin
// kotlin：
class KtDemo @JvmOverloads constructor(a:String,b:String,c:Int = 100){
......
}
//java
 public static void main(String[] args) {
       KtDemo demo1  = new KtDemo("a","b");
       KtDemo demo2  = new KtDemo("a","b",200);
    }
```

此处需要注意：

<font color = "ff0000" size = "4">android自定义View时，默认的DefStyleAttr！</font>

例如：

```kotlin
class DemoView @JvmOverloads constructor(
        context: Context, 
        attrs: AttributeSet? = null, 
        defStyleAttr: Int = 0
) : AppCompatEditText(context, attrs, defStyleAttr) {
}
```

可能出现的问题：

​	在 EditView 的场景下，你会发现焦点没有了，点击之后软键盘也不会自动弹出。

> 原因就在 AS 在自动生成的代码时，对参数默认值的处理。
>
> 当在自定义 View 时，通过 AS 生成重载方法时，它对参数默认值的处理规则是这样的。
>
> 1. 遇到对象，默认值为 null。
> 2. 遇到基础数据类型，默认值为基本数据类型的默认值。例如 Int 就是 0，Boolean 就是 false。
>
> 而在这里的场景下， `defStyleAttr` 这个参数的类型为 Int，所以默认值会被赋值为 0，但是它并不是需要的。

在 Android 中，当 View 通过 XML 文件来布局使用时，会调用两个参数的构造方法 `(Context context, AttributeSet attrs)`，而它内部会调用三个参数的构造方法，并传递一个默认的 `defStyleAttr`，注意它并不是 0。

`AppCompatEditText` 中的实现：

```java
public AppCompatEditText(Context context, 
                         AttributeSet attrs) {
    this(context, attrs, R.attr.editTextStyle);
}
```

再修改 DemoView 中对 `defStyleAttr` 默认值的指定即可。

```kotlin
class DemoView @JvmOverloads constructor(
        context: Context,
        attrs: AttributeSet? = null, 
        defStyleAttr: Int = R.attr.editTextStyle
) : AppCompatEditText(context, attrs, defStyleAttr) {
}
```

### 6.方法返回值为多个参数

利用对返回值对象的解构函数，可返回多个参数，kotlin 包下提供好的有：

```kotlin

package kotlin


/**
 * Represents a generic pair of two values.
 *
 * There is no meaning attached to values in this class, it can be used for any purpose.
 * Pair exhibits value semantics, i.e. two pairs are equal if both components are equal.
 *
 * An example of decomposing it into values:
 * @sample samples.misc.Tuples.pairDestructuring
 *
 * @param A type of the first value.
 * @param B type of the second value.
 * @property first First value.
 * @property second Second value.
 * @constructor Creates a new instance of Pair.
 */
public data class Pair<out A, out B>(
    public val first: A,
    public val second: B
) : Serializable {

    /**
     * Returns string representation of the [Pair] including its [first] and [second] values.
     */
    public override fun toString(): String = "($first, $second)"
}

/**
 * Creates a tuple of type [Pair] from this and [that].
 *
 * This can be useful for creating [Map] literals with less noise, for example:
 * @sample samples.collections.Maps.Instantiation.mapFromPairs
 */
public infix fun <A, B> A.to(that: B): Pair<A, B> = Pair(this, that)

/**
 * Converts this pair into a list.
 * @sample samples.misc.Tuples.pairToList
 */
public fun <T> Pair<T, T>.toList(): List<T> = listOf(first, second)

/**
 * Represents a triad of values
 *
 * There is no meaning attached to values in this class, it can be used for any purpose.
 * Triple exhibits value semantics, i.e. two triples are equal if all three components are equal.
 * An example of decomposing it into values:
 * @sample samples.misc.Tuples.tripleDestructuring
 *
 * @param A type of the first value.
 * @param B type of the second value.
 * @param C type of the third value.
 * @property first First value.
 * @property second Second value.
 * @property third Third value.
 */
public data class Triple<out A, out B, out C>(
    public val first: A,
    public val second: B,
    public val third: C
) : Serializable {

    /**
     * Returns string representation of the [Triple] including its [first], [second] and [third] values.
     */
    public override fun toString(): String = "($first, $second, $third)"
}

/**
 * Converts this triple into a list.
 * @sample samples.misc.Tuples.tripleToList
 */
public fun <T> Triple<T, T, T>.toList(): List<T> = listOf(first, second, third)

```

例子：

```kotlin
 //onCreate:
 val (transition, scene1, scene2) = initTransXML()
// function
 private fun initTransXML(): Triple<TransitionManager, Scene, Scene> {
        val transitionInflater = TransitionInflater.from(this)
        val transition =
            transitionInflater.inflateTransitionManager(R.transition.trans_manager, mContainer)

        val scene1 = Scene.getSceneForLayout(mContainer, R.layout.layout_translation_before, this)
        val scene2 = Scene.getSceneForLayout(mContainer, R.layout.layout_translation_after, this)
        return Triple(transition, scene1, scene2)
    }
```

