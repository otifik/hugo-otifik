---
title: "Kotlin反射&泛型"
date: 2023-05-17T09:49:52+08:00
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
- 泛型
- kotlin
- java
---

泛型擦除机制：编译器会将T这样的泛型擦除成Object，虚拟机本身是不知道泛型的。Java的泛型是由编译器在编译时实行的，编译器内部永远把所有类型T视为Object处理，但是，在需要转型的时候，编译器会根据T的类型自动为我们实行安全地强制转型。

编译器看到的代码：
```java
public class Foo<T>{
    private T bar;

    public Foo(T bar) {
        this.bar = bar;
    }

    public T getBar() {
        return bar;
    }

    public void setBar(T bar) {
        this.bar = bar;
    }
}
```
虚拟机看到的代码：
```java
public class Foo{
    private Object bar;

    public Foo(Object bar) {
        this.bar = bar;
    }

    public Object getBar() {
        return bar;
    }

    public void setBar(Object bar) {
        this.bar = bar;
    }
}
```

## extends通配符/协变


### java extends通配符
\<? extends T>允许调用读方法T get()获取T的引用，但不允许调用写方法set(T)传入T的引用（传入null除外）；协变。out，类似于fun(T: Number)。

这段代码中，Foo是一个泛型类型，有一个bar参数，Main类中的static方法reset包含一个Foo<Number>类型的参数f。main函数运行，创建一个Foo<Integer>类型的变量传入reset中，虽然Integer是Number的子类，但显然Foo<Number>并不是Foo<Integer>的子类，但是如果能编译通过，reset函数中f调用getBar得到一个Integer类型的值赋值给Number是完全可行的。

```java
public class Main {

    public static void main(String[] args) {
        Foo<Integer> f = new Foo<Integer>(100);
        int res = reset(f); //'reset(Foo<java.lang.Number>)' in 'Main' cannot be applied to '(Foo<java.lang.Integer>)'
        System.out.println(res);

    }

    static int reset(Foo<Number> f){
        Number bar = f.getBar(); 
        return bar.intValue();
    }

}

class Foo<T>{
    private T bar;

    public Foo(T bar) {
        this.bar = bar;
    }

    public T getBar() {
        return bar;
    }

    public void setBar(T bar) {
        this.bar = bar;
    }
}
```

所以我们可以使用<? extends T>这样的上界通配符，来让“Foo<Integer>成为Foo<Number>的子类型”，除此之外可以传入Foo<Double>、Foo<Float>等类型。

```java
public class Main {

    public static void main(String[] args) {
        Foo<Integer> f = new Foo<Integer>(100);
        int res = reset(f);
        System.out.println(res);

    }

    static int reset(Foo<? extends Number> f){
        Number bar = f.getBar();
        return bar.intValue();
    }

}

class Foo<T>{
    private T bar;

    public Foo(T bar) {
        this.bar = bar;
    }

    public T getBar() {
        return bar;
    }

    public void setBar(T bar) {
        this.bar = bar;
    }
}
```

但是需要注意的是，reset中不能调用setBar函数，因为f的类型是不明确的，是Foo<? extends Number>，它既可能是符合要求的Foo<Integer>，也可能是Foo<Double>，类型转换会出现异常。如下：
```java
public class Main {

    public static void main(String[] args) {
        Foo<Integer> f = new Foo<Integer>(100);
        int res = reset(f);
        System.out.println(res);

    }

    static int reset(Foo<? extends Number> f){
        Number bar = f.getBar();
        f.setBar(new Integer(1)); //报错：'setBar(capture<? extends java.lang.Number>)' in 'Foo' cannot be applied to '(java.lang.Integer)'
        return f.getBar().intValue();
    }

}

class Foo<T>{
    private T bar;

    public Foo(T bar) {
        this.bar = bar;
    }

    public T getBar() {
        return bar;
    }

    public void setBar(T bar) {
        this.bar = bar;
    }
}
```
**这是<? extends T>的一个重要限制，set<? extends T>方法无法传递任何T的子类型给set<? extends T>。**
通俗易懂的解释，我可以去拿这个里面的东西，但是想要改变这个东西的话，由于我不知道他是什么类型的，一旦进行改变就可能会出现类型转换异常。
我拥有子类的能力去取东西，但我要去改变的时候，我不知道他的类型是什么。

这里唯一的例外是可以给方法参数传入null：
```java
f.setBar(null); // ok, 但是后面会抛出NullPointerException
f.getBar(); // NullPointerException
```

### kotlin 协变

kotlin中的泛型协变亦是如此。
上面的java例子改造成kotlin，如下：
```kotlin
fun main() {
    val f = Foo(100)
    val res = reset(f) //报错：Type mismatch: inferred type is Foo<Int> but Foo<Number> was expected
    println(res)
}

fun reset(f: Foo<Number>): Int{
    val bar = f.bar
    return bar.toInt()
}

class Foo<T>(var bar: T)
```
这时kotlin可以利用out关键字进行泛型的协变。

```kotlin
fun main() {
    val f = Foo(100)
    val res = reset(f)
    println(res)
}

fun reset(f: Foo<out Number>): Int{
    val bar = f.bar
    return bar.toInt()
}

class Foo<T>(var bar: T)
```
out从字面意思上来理解就是只能拿取获得，可以读不可写的意思，对应get，报错原因同上，这里不在进行解释了。


## super通配符/逆变

### java super通配符

\<? super T>允许调用写方法set(T)传入T的引用，但不允许调用读方法T get()获取T的引用（获取Object除外）。逆变。in


这段代码中，Foo是一个泛型类型，有一个bar参数，Main类中的static方法reset包含一个Foo<Integer>类型的参数f和一个Integer类型的i。main函数运行，创建一个Foo<Integer>类型的变量f1传入reset中，正常可以通过编译；接着创建一个Foo<Number>类型的变量f2传入reset中，可以发现编译器报错了，原因是类型不匹配，但假如这段代码编译能够通过，Foo<Number>类型的f2变量完全可以调用setBar来把Number类型的bar值设置为一个Integer类型，因为Integer类型是Number类型的子类。

```java
public class Main {

    public static void main(String[] args) {
        Foo<Integer> f1 = new Foo<Integer>(100);
        Foo<Number> f2 = new Foo<>(200);
        reset(f1,new Integer(1));
        reset(f2,new Integer(1)); //报错：'reset(Foo<java.lang.Integer>, java.lang.Integer)' in 'Main' cannot be applied to '(Foo<java.lang.Number>, java.lang.Integer)'
    }

    static void reset(Foo<Integer> f,Integer i){
        f.setBar(i);
    }

}

class Foo<T>{
    private T bar;

    public Foo(T bar) {
        this.bar = bar;
    }

    public T getBar() {
        return bar;
    }

    public void setBar(T bar) {
        this.bar = bar;
    }
}
```

所以我们可以使用<? super T>这样的通配符，让“Foo<Number>成为Foo<Integer>的子类”。
```java
public class Main {

    public static void main(String[] args) {
        Foo<Integer> f1 = new Foo<Integer>(100);
        Foo<Number> f2 = new Foo<>(200);
        reset(f1,new Integer(1));
        reset(f2,new Integer(1));

    }

    static void reset(Foo<? super Integer> f,Integer i){
        f.setBar(i);
    }

}

class Foo<T>{
    private T bar;

    public Foo(T bar) {
        this.bar = bar;
    }

    public T getBar() {
        return bar;
    }

    public void setBar(T bar) {
        this.bar = bar;
    }
}
```
但是需要注意的是，reset中不能调用getBar函数，因为f的类型是不明确的，是Foo<? super Integer>，它既可能是的Foo<Number>，也可能是Foo<Object>，类型转换会出现异常。如下：
```java
public class Main {

    public static void main(String[] args) {
        Foo<Integer> f1 = new Foo<Integer>(100);
        Foo<Number> f2 = new Foo<>(200);
        reset(f1,new Integer(1));
        reset(f2,new Integer(1));

    }

    static void reset(Foo<? super Integer> f,Integer i){
        f.setBar(i);
        Integer bar = f.getBar(); //报错：Incompatible types. Found: 'capture<? super java.lang.Integer>', required: 'java.lang.Integer'
    }

}

class Foo<T>{
    private T bar;

    public Foo(T bar) {
        this.bar = bar;
    }

    public T getBar() {
        return bar;
    }

    public void setBar(T bar) {
        this.bar = bar;
    }
}
```

我拥有父类的能力去给改变东西，但如果我自己拿一个东西，我不知道他的类型是什么。
但有个特殊情况，唯一可以接收getFirst()方法返回值的是Object类型，如下：
```java
Object bar = f.getBar();
```

### kotlin 逆变

kotlin中亦是如此，不做过多阐述：

```kotlin
fun main() {
    val f1 = Foo<Int>(100)
    val f2 = Foo<Number>(200)
    
    reset(f1,1)
    reset(f2,1) //报错：Type mismatch: inferred type is Foo<Number> but Foo<Int> was expected

}

fun reset(f: Foo<Int>,i: Int){
    f.bar = i
}

class Foo<T>(var bar: T)
```
增加in关键字，进行逆变
```kotlin
fun main() {
    val f1 = Foo<Int>(100)
    val f2 = Foo<Number>(200)

    reset(f1,1)
    reset(f2,1)

}

fun reset(f: Foo<in Int>,i: Int){
    f.bar = i
}

class Foo<T>(var bar: T)
```
in从字面意思上来理解就是只能给予，可以写不可读的意思，对应out。


PECS原则：Producer Extends Consumer Super。即：如果需要返回T，它是生产者（Producer），要使用extends通配符/out；如果需要写入T，它是消费者（Consumer），要使用super通配符/in。


q1:
java中为什么Foo<Number>和Foo<Integer>没有继承关系？Number不是Integer的父类吗？

a1:
在Java中，Pair<Number>和Pair<Integer>之间没有继承关系。虽然Number是Integer的父类，但是泛型不支持协变（covariance）或逆变（contravariance）的类型参数继承。

```java
Foo<Number> numberFoo = new Foo<>(10);
Foo<Integer> integerFoo = new Foo<>(20);
numberFoo = integerFoo; // 编译错误
```
在这个例子中，如果Foo<Number>和Foo<Integer>之间存在继承关系，那么将integerFoo赋值给numberFoo应该是合法的。然而，Java中的泛型是不可变的（invariant），这意味着即使Integer是Number的子类型，Foo<Number>和Foo<Integer>之间没有继承关系，无法进行赋值。