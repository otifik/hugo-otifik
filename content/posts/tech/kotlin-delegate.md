---
title: "Kotlin委托深究"
date: 2023-05-15T15:05:34+08:00
author: ["otifik"] #作者
draft: false
description: "" #描述
showToc: true # 显示目录
TocOpen: true # 自动展开目录
disableShare: true # 底部不显示分享栏
weight: # 输入1可以顶置文章，用来给文章展示排序，不填就默认按时间排序
cover:
    image: "" #图片路径：posts/tech/文章1/picture.png
    caption: "" #图片底部描述
    alt: ""
    relative: false
tags: 
- kotlin
- 委托
---

Kotiln的委托，简而言之，就是把让我干的事交给别人做。分为类委托和属性委托。
用于解耦代码。

引用文章：
https://blog.csdn.net/willway_wang/article/details/120795321
https://www.kotlincn.net/docs/reference/delegated-properties.html

# 类委托
将接口的实现委托给另一个对象
但值得注意的是**类必须实现一个接口，委托类必须是类所实现接口的子类型**。
```kotlin
interface Base {
    fun print()
}

class BaseImpl(val x: Int) : Base {
    override fun print() { print(x) }
}

class Derived(b: Base) : Base by b

fun main() {
    val b = BaseImpl(10)
    Derived(b).print()
}
```
kotlin转java字节码解析：
by关键字自动生成了Derived的print，相当于Derived让Base的一个实现类帮它调用了print函数。
![](kotlinbytecode1.png)

委托类的接口实现可以通过override覆写，如：
```kotlin
interface Base {
    fun print()
}

class BaseImpl(val x: Int) : Base {
    override fun print() { print(x) }
}

class Derived(b: Base) : Base by b {
    override fun print() {
        print("fun has override")
    }
}

fun main() {
    val b = BaseImpl(10)
    Derived(b).print()
}
```
kotlin转java字节码解析：
Derived覆写了Base实现类的print
![](kotlinbytecode2.png)


# 属性委托
将属性访问器的实现委托给另一个对象


```kotlin
class Example {
    var p: String by Delegate()
}

class Delegate : ReadWriteProperty<Any, String> {

    override fun getValue(thisRef: Any, property: KProperty<*>): String {
        return "$thisRef, thank you for delegating '${property.name}' to me!"
    }

    override fun setValue(thisRef: Any, property: KProperty<*>, value: String) {
        println("$value has been assigned to '${property.name}' in $thisRef.")
    }
}

fun main() {
    val e = Example()
    println(e.p)
}
```
解析：
原理和类委托大致类似

①
```kotlin
class Example {
    var p: String by Delegate()
}
```
其中这段代码可以等价为：
```kotlin
class Example {
    private val delegate = Delegate()
    var p: String
    	set(value: String) = delegate.setValue(this, ..., value)
    	get() = delegate.getValue(this, ...)
}
```
可以看到其实p这个属性的get/set交给了delegate来进行维护
**by**关键字的作用是对它后面的表达式求值（比方说lazy函数）来获取这个对象，在这里就是获取到了Delegate对象。
编译器会创建一个隐藏的辅助属性，并使用委托对象的实例对它进行初始化，在这里就是把Delegate对象赋值给了delegate属性。

kotlin转java字节码解析
```java
public final class Example {
   // $FF: synthetic field
   static final KProperty[] $$delegatedProperties = new KProperty[]{(KProperty)Reflection.mutableProperty1(new MutablePropertyReference1Impl(Example.class, "p", "getP()Ljava/lang/String;", 0))};
   @NotNull
   private final Delegate p$delegate = new Delegate();

   @NotNull
   public final String getP() {
      return this.p$delegate.getValue(this, $$delegatedProperties[0]);
   }

   public final void setP(@NotNull String var1) {
      Intrinsics.checkNotNullParameter(var1, "<set-?>");
      this.p$delegate.setValue(this, $$delegatedProperties[0], (String)var1);
   }
}

public final class Delegate implements ReadWriteProperty {
   @NotNull
   public String getValue(@NotNull Object thisRef, @NotNull KProperty property) {
      Intrinsics.checkNotNullParameter(thisRef, "thisRef");
      Intrinsics.checkNotNullParameter(property, "property");
      return thisRef + ", thank you for delegating '" + property.getName() + "' to me!";
   }

   // $FF: synthetic method
   // $FF: bridge method
   public Object getValue(Object var1, KProperty var2) {
      return this.getValue(var1, var2);
   }

   public void setValue(@NotNull Object thisRef, @NotNull KProperty property, @NotNull String value) {
      Intrinsics.checkNotNullParameter(thisRef, "thisRef");
      Intrinsics.checkNotNullParameter(property, "property");
      Intrinsics.checkNotNullParameter(value, "value");
      String var4 = value + " has been assigned to '" + property.getName() + "' in " + thisRef + '.';
      System.out.println(var4);
   }

   // $FF: synthetic method
   // $FF: bridge method
   public void setValue(Object var1, KProperty var2, Object var3) {
      this.setValue(var1, var2, (String)var3);
   }
}
public final class AKt {
   public static final void main() {
      Example e = new Example();
      String var1 = e.getP();
      System.out.println(var1);
   }

   // $FF: synthetic method
   public static void main(String[] var0) {
      main();
   }
}
```
其中thisRef代表发起委托的对象，**$$delegatedProperties**对象，它是一个KProperty类型的数组，它的元素封装了类，属性名，getter方法签名等信息，也会传递给委托类；KProperty 主要是用来封装属性的元信息，提供给委托类使用，比如在委托类的 setValue 方法中通知属性发生变化时，就会用到 KProperty 里的属性名信息了。

②
```kotlin
class Delegate {
    operator fun getValue(thisRef: Any?, property: KProperty<*>): String {
        return "$thisRef, thank you for delegating '${property.name}' to me!"
    }

    operator fun setValue(thisRef: Any?, property: KProperty<*>, value: String) {
        println("$value has been assigned to '${property.name}' in $thisRef.")
    }
}
```
委托类必须具有**getValue**和**setValue**方法（如果是可变属性的话）,定义在ReadWriteProperty接口里
ReadOnlyProperty对应val，ReadWriteProperty对应var
```kotlin
public fun interface ReadOnlyProperty<in T, out V> {
    public operator fun getValue(thisRef: T, property: KProperty<*>): V
}

public interface ReadWriteProperty<in T, V> : ReadOnlyProperty<T, V> {
    public override operator fun getValue(thisRef: T, property: KProperty<*>): V
    public operator fun setValue(thisRef: T, property: KProperty<*>, value: V)
}
```

虽然 Kotlin 提供了 ReadWriteProperty 和 ReadOnlyProperty 封装了约定的方法给我们使用，但是当我们定义委托类时并不是一定要实现 Kotlin 提供的接口。

实际上，只要保持委托类里的 setValue 和 getValue 方法与约定的 setValue 方法和 getValue 方法一致就可以了。
如
```kotlin
import kotlin.reflect.KProperty
import kotlin.reflect.full.valueParameters

class Example {
    var p: String by Delegate()
}

class Delegate {
    operator fun getValue(thisRef: Any?, property: KProperty<*>): String {
        return "$thisRef, thank you for delegating '${property.name}' to me!"
    }

    operator fun setValue(thisRef: Any?, property: KProperty<*>, value: String) {
        println("$value has been assigned to '${property.name}' in $thisRef.")
    }
}

fun main(){
    val e = Example()
    println(e.p)
}
```

## kotlin内置委托属性

### lazy
lazy()是接受一个lambda并返回一个Lazy<T>实例的函数，返回的实例可以作为实现延迟属性的委托：第一次调用get()会执行已传递给lazy()的lambda表达式并记录结果，后续调用get()只是返回记录的结果。
```kotlin
public actual fun <T> lazy(initializer: () -> T): Lazy<T> = SynchronizedLazyImpl(initializer)

public interface Lazy<out T> {
    /**
     * Gets the lazily initialized value of the current Lazy instance.
     * Once the value was initialized it must not change during the rest of lifetime of this Lazy instance.
     */
    public val value: T

    /**
     * Returns `true` if a value for this Lazy instance has been already initialized, and `false` otherwise.
     * Once this function has returned `true` it stays `true` for the rest of lifetime of this Lazy instance.
     */
    public fun isInitialized(): Boolean
}

public inline operator fun <T> Lazy<T>.getValue(thisRef: Any?, property: KProperty<*>): T = value

private class SynchronizedLazyImpl<out T>(initializer: () -> T, lock: Any? = null) : Lazy<T>, Serializable {
    private var initializer: (() -> T)? = initializer
    @Volatile private var _value: Any? = UNINITIALIZED_VALUE
    // final field is required to enable safe publication of constructed instance
    private val lock = lock ?: this

    override val value: T
        get() {
            val _v1 = _value
            if (_v1 !== UNINITIALIZED_VALUE) {
                @Suppress("UNCHECKED_CAST")
                return _v1 as T
            }

            return synchronized(lock) {
                val _v2 = _value
                if (_v2 !== UNINITIALIZED_VALUE) {
                    @Suppress("UNCHECKED_CAST") (_v2 as T)
                } else {
                    val typedValue = initializer!!()
                    _value = typedValue
                    initializer = null
                    typedValue
                }
            }
        }

    override fun isInitialized(): Boolean = _value !== UNINITIALIZED_VALUE

    override fun toString(): String = if (isInitialized()) value.toString() else "Lazy value not initialized yet."

    private fun writeReplace(): Any = InitializedLazyImpl(value)
}
```
通过源码可以看出，Lazy.kt中定义了符合约定的扩展函数getValue，实例化SynchronizedLazyImpl就是委托对象。

## 可观察属性 Observable

Delegates.observable() 接受两个参数：初始值与修改时处理程序（handler）。 每当我们给属性赋值时会调用该处理程序（在赋值后执行）。它有三个参数：被赋值的属性、旧值与新值：
```kotlin
var name: String by Delegates.observable("no name") {
    prop, old, new ->
    println("$old -> $new")
}
```
源码
```kotlin
public inline fun <T> observable(initialValue: T, crossinline onChange: (property: KProperty<*>, oldValue: T, newValue: T) -> Unit): ReadWriteProperty<Any?, T> =
    object : ObservableProperty<T>(initialValue) {
        override fun afterChange(property: KProperty<*>, oldValue: T, newValue: T) = onChange(property, oldValue, newValue)
    }

public abstract class ObservableProperty<V>(initialValue: V) : ReadWriteProperty<Any?, V> {
    private var value = initialValue

    /**
     *  The callback which is called before a change to the property value is attempted.
     *  The value of the property hasn't been changed yet, when this callback is invoked.
     *  If the callback returns `true` the value of the property is being set to the new value,
     *  and if the callback returns `false` the new value is discarded and the property remains its old value.
     */
    protected open fun beforeChange(property: KProperty<*>, oldValue: V, newValue: V): Boolean = true

    /**
     * The callback which is called after the change of the property is made. The value of the property
     * has already been changed when this callback is invoked.
     */
    protected open fun afterChange(property: KProperty<*>, oldValue: V, newValue: V): Unit {}

    public override fun getValue(thisRef: Any?, property: KProperty<*>): V {
        return value
    }

    public override fun setValue(thisRef: Any?, property: KProperty<*>, value: V) {
        val oldValue = this.value
        if (!beforeChange(property, oldValue, value)) {
            return
        }
        this.value = value
        afterChange(property, oldValue, value)
    }
}
```

## 委托给另一个属性

从 Kotlin 1.4 开始，一个属性可以把它的 getter 与 setter 委托给另一个属性。这种委托对于顶层和类的属性（成员和扩展）都可用。该委托属性可以为：

-顶层属性
-同一个类的成员或扩展属性
-另一个类的成员或扩展属性
为将一个属性委托给另一个属性，应在委托名称中使用恰当的::限定符，例如，this::delegate或MyClass::delegate。

```kotlin
var topLevelInt: Int = 0

class ClassWithDelegate(val anotherClassInt: Int)
class MyClass(var memberInt: Int, val anotherClassInstance: ClassWithDelegate) {
    var delegatedToMember: Int by this::memberInt
    var delegatedToTopLevel: Int by ::topLevelInt
    
    val delegatedToAnotherClass: Int by anotherClassInstance::anotherClassInt
}
var MyClass.extDelegated: Int by ::topLevelInt
```

使用场景：当想要以一种向后兼容的方式重命名一个属性的时候：引入一个新的属性、 使用 @Deprecated 注解来注解旧的属性、并委托其实现。
```kotlin
class MyClass {
   var newName: Int = 0
   @Deprecated("Use 'newName' instead", ReplaceWith("newName"))
   var oldName: Int by this::newName
}

fun main() {
   val myClass = MyClass()
   // 通知：'oldName: Int' is deprecated.
   // Use 'newName' instead
   myClass.oldName = 42
   println(myClass.newName) // 42
}
```

## 将属性储存在映射中
一个常见的用例是在一个映射（map）里存储属性的值。这经常出现在像解析JSON或者做其他“动态”事情的应用中。在这种情况下，你可以使用映射实例自身作为委托来实现委托属性。
```kotlin
class User(val map: Map<String, Any?>) {
    val name: String by map
    val age: Int     by map
}
```
在这个例子中，构造函数接受一个映射参数：
```kotlin
val user = User(mapOf(
    "name" to "John Doe",
    "age"  to 25
))
```
委托属性会从这个映射中取值（通过字符串键——属性的名称）：
```kotlin
println(user.name) // Prints "John Doe"
println(user.age)  // Prints 25
```
这也适用于 var 属性，如果把只读的 Map 换成 MutableMap 的话：
```kotlin
class MutableUser(val map: MutableMap<String, Any?>) {
    var name: String by map
    var age: Int     by map
}
```
## 局部委托属性
你可以将局部变量声明为委托属性。 例如，你可以使一个局部变量惰性初始化：
```kotlin
fun example(computeFoo: () -> Foo) {
    val memoizedFoo by lazy(computeFoo)
​
    if (someCondition && memoizedFoo.isValid()) {
        memoizedFoo.doSomething()
    }
}
```
memoizedFoo 变量只会在第一次访问时计算。 如果 someCondition 失败，那么该变量根本不会计算。

## 提供委托
通过定义 provideDelegate 操作符，可以扩展创建属性实现所委托对象的逻辑。 如果 by 右侧所使用的对象将 provideDelegate 定义为成员或扩展函数，那么会调用该函数来 创建属性委托实例。

provideDelegate 的一个可能的使用场景是在创建属性时（而不仅在其 getter 或 setter 中）检查属性一致性。

```kotlin
class ResourceDelegate<T> : ReadOnlyProperty<MyUI, T> {
    override fun getValue(thisRef: MyUI, property: KProperty<*>): T { ... }
}
    
class ResourceLoader<T>(id: ResourceID<T>) {
    operator fun provideDelegate(
            thisRef: MyUI,
            prop: KProperty<*>
    ): ReadOnlyProperty<MyUI, T> {
        checkProperty(thisRef, prop.name)
        // 创建委托
        return ResourceDelegate()
    }

    private fun checkProperty(thisRef: MyUI, name: String) { …… }
}

class MyUI {
    fun <T> bindResource(id: ResourceID<T>): ResourceLoader<T> { …… }

    val image by bindResource(ResourceID.image_id)
    val text by bindResource(ResourceID.text_id)
}
```
prop.name，此处prop指的是被委托属性对象，因此prop.name就是属性的名称，在 checkProperty(thisRef, prop.name)中即可自行检查属性一致性，检查无误后，即可通过返回的getValue完成属性委托