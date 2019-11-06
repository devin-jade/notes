---
layout: post
title: Java String
categories: [Java Base]
tags: String
---

# Java String

本文基于JDK8

常量： **内存地址不变, 则值也不可以改变的东西称为常量**，  **我们可以通过final关键字来定义常量**。

变量： **内存地址不变,值可以改变的东西称为变量**。

## String的不可变性：

### 不可变对象：

**如果一个对象，在它创建完成之后，不能再改变它的状态，那么这个对象就是不可变的。不能改变状态指的是不能改变对象内的成员变量，包括：**

1. 基本数据类型的值不能改变;
2. 引用类型的变量不能指向其他的对象;
3. 引用类型指向的对象的状态也不能改变;

另外：

- **除了构造函数之外，不应该有其它任何函数（至少是任何public函数）修改任何成员变量;**
- **任何使成员变量获得新值的函数**都应该**将新的值保存在新的对象中，而保持原来的对象不被修改。**

### String 不可变

String类是不可变类 (基本类型的包装类都是不可改变的) 的典型代表，也是Immutable设计模式的典型应用。String变量一旦初始化后就不能更改，禁止改变对象的状态，从而增加共享对象的坚固性、减少对象访问的错误，同时还避免了在多线程共享时进行同步的需要。

![](https://raw.githubusercontent.com/devin-jade/devin-imag/master/web/20191104003226.png)

```java
// JDK8
public final class String
    implements java.io.Serializable, Comparable<string>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];

    /** Cache the hash code for the string */
    private int hash; // Default to 0</string>
```

>- final class 保证类不能被继承，被重写；
>- final char 保证char引用不可以被修改;
>- 不提供value的访问方法，保证value引用的数组对象不可以变

```java
public String substring(int beginIndex, int endIndex) {
        if (beginIndex < 0) {
            throw new StringIndexOutOfBoundsException(beginIndex);
        }
        if (endIndex > value.length) {
            throw new StringIndexOutOfBoundsException(endIndex);
        }
        int subLen = endIndex - beginIndex;
        if (subLen < 0) {
            throw new StringIndexOutOfBoundsException(subLen);
        }
        return ((beginIndex == 0) && (endIndex == value.length)) ? this
                : new String(value, beginIndex, subLen);
    }
```

>subString 时，会new一个字符串对象然后返回，原始的value不变。
>
>replace， replaceAll， toLowerCase 以及字符串拼接都是相同的思路（基本类型的包装类也是这种思路）。

通过上面的措施保证String的状态不可修改，进而得到不可变性。

### 反射其实可以修改String

因为反射可以修改成员变量的访问权限，因此其实String的不可以变性是可以被打破的；只是一般情况我们都不这样做。

```java
public static void testModifyString() throws Exception {
    //创建字符串"Hello World"， 并赋给引用s
    String s = "Hello World"; 
    System.out.println("s = " + s); //Hello World

    //获取String类中的value字段
    Field valueFieldOfString = String.class.getDeclaredField("value");
    //改变value属性的访问权限
    valueFieldOfString.setAccessible(true);

    //获取s对象上的value属性的值
    char[] value = (char[]) valueFieldOfString.get(s);
    //改变value所引用的数组中的第5个字符
    value[5] = '_';

    System.out.println("s = " + s);  //Hello_World
}
```

## String的创建

### 字面值形式

 判断这个常量是否存在于常量池，
  如果存在，
   判断这个常量是存在的引用还是常量，
    如果是引用，返回引用地址指向的堆空间对象，
    如果是常量，则直接返回常量池常量，
  如果不存在，
    在常量池中创建该常量，并返回此常量 

字面值形式创建的字符串，在编译期间会得到最终的结果；

```java
String a1 = "AA";//在常量池上创建常量AA
String a2 = "AA";//直接返回已经存在的常量AA
System.out.println(a1 == a2); //true
String a2 = "AABB";
String a3 = "AA" + "BB";
System.out.println(a2 == a3); //true

String a3 = new String("AA");    //在堆上创建对象AA 在常量池上创建"AA"
a3.intern(); //在常量池上创建"AA"
String a4 = "AA"; //由于常量池已经存在AA，所以a4是常量池的AA
System.out.println(a3 == a4); //false,a3指向堆对象，a4指向常量池
System.out.println(a3.intern == a4); //ture，a3.intern返回的就是常量池地址，所以true
```

### new 关键字创建

  首先在堆上创建对象(无论堆上是否存在相同字面量的对象),
 然后判断常量池上是否存在字符串的字面量，
  如果不存在
   在常量池上创建常量
  如果存在
   不做任何操作 

```java
String a44 = new String("AABB"); //在堆上创建对象AABB，在常量池上创建常量AABB
a44.intern(); // 返回常量池的AABB
String a45 = "AA" + "BB"; //编译阶段a45=AABB，常量池有，a45就是常量池AABB
System.out.println(a44 == a45); //a44是堆上的，a45是常量池的
System.out.println(a44.intern == a45); //a44是堆上的，a45是常量池的

String a14 = new String("AA") + new String("BB"); //堆上创建对象AA、BB和AABB，常量池创建AA和BB，由于AABB不是通过字面量字符的方式定义出来的，因此常量池不会创建AABB，只会添加一个引用指向堆上的AABB
a14.intern(); // 在常量池创建AABB的引用，指向堆上的AABB
String a15 = "AA" + "BB";// 编译阶段a15=AABB,常量池有引用，a15指向了堆上的AABB
System.out.println(a14 == a15); //true  都是指向堆上的AABB
System.out.println(a14.intern() == a15); //true 都是指向堆上的AABB
System.out.println(a14 == a15.intern()); //true 都是指向堆上的AABB
System.out.println(a14.intern() == a15.intern()); //true 都是指向堆上的AABB

       
```

### 字面值+new 拼接

 首先创建两个对象，一个是new String的对象，一个是相加后的对象
 然后判断双引号常量与new String的字面量在常量池是否存在
  如果存在
   不做操作
  如果不存在
   则在常量池上创建对象的常量 

```java
String a1 = "AABB";
String a2 = "AA" + new String("BB");
System.out.println(a1 == a2.intern());//true
System.out.println(a2 == a2.intern()); //false
```



## 字符串池

字符串的分配，和其他的对象分配一样，耗费高昂的时间与空间代价。JVM为了提高性能和减少内存开销，在实例化字符串字面值的时候进行了一些优化。为了减少在JVM中创建的字符串的数量，字符串类维护了一个**字符串常量池**，每当以字面值形式创建一个字符串时，JVM会首先检查字符串常量池：如果字符串已经存在池中，就返回池中的实例引用；如果字符串不在池中，就会实例化一个字符串，或者创建一个引用并放到池中。**Java能够进行这样的优化是因为字符串是不可 变的，可以不用担心数据冲突进行共享**.

### String.intern

 判断这个常量是否存在于常量池。
  如果存在
   判断存在内容是引用还是常量，
    如果是引用，
     返回引用地址指向堆空间对象，
    如果是常量，
     直接返回常量池常量
  如果不存在，
   将当前对象引用复制到常量池,并且返回的是当前对象的引用 

```java
String a1 = "AA";
System.out.println(a1 == a1.intern()); //true
String a2 = new String("B") + new String("B");
a2.intern();
String a3 = new String("B") + new String("B");
System.out.println(a2 == a3.intern());//true
System.out.println(a3 == a3.intern());//false
String a4 = new String("C") + new String("C");
System.out.println(a4 == a4.intern()); //true
```

### String、StringBuilder、StringBuffer

由于String是不可变的，每次改变字符串的时候，其实是创建了一个新的字符串，然后改变指针的指向，对于频繁的创建字符串会影响性能，而且会造成内存的浪费，引起频繁GC；多以java 提供了两个工具类来处理易变的字符串。

StringBuffer 与 StringBuilder

　　首先需要明确的是，StringBuffer 始于 JDK 1.0，而 StringBuilder 始于 JDK 5.0；此外，从 JDK 1.5 开始，对含有字符串变量 (非字符串字面值) 的连接操作(+)，JVM 内部是采用 StringBuilder 来实现的，而在这之前，这个操作是采用 StringBuffer 实现的。

　　JDK的实现中 StringBuffer 与 StringBuilder 都继承自 AbstractStringBuilder。AbstractStringBuilder的实现原理为：AbstractStringBuilder中采用一个 char数组 来保存需要append的字符串，char数组有一个初始大小，当append的字符串长度超过当前char数组容量时，则对char数组进行动态扩展，即重新申请一段更大的内存空间，然后将当前char数组拷贝到新的位置，因为重新分配内存并拷贝的开销比较大，所以每次重新申请内存空间都是采用申请大于当前需要的内存空间的方式，这里是 2 倍。

　　StringBuffer 和 StringBuilder 都是可变的字符序列，但是二者最大的一个不同点是：StringBuffer 是线程安全的，而 StringBuilder 则不是。StringBuilder 提供的API与StringBuffer的API是完全兼容的，即，StringBuffer 与 StringBuilder 中的方法和功能完全是等价的，但是后者一般要比前者快。因此，可以这么说，StringBuilder 的提出就是为了在单线程环境下替换 StringBuffer 。

　　在单线程环境下，优先使用 StringBuilder。

## 字符串比较

 使用equals方法 ： **先比较引用是否相同(是否是同一对象)，再检查是否为同一类型（str instanceof String）， 最后比较内容是否一致（String 的各个成员变量的值或内容是否相同）。****这也同样适用于诸如 Integer 等的八种包装器类。** 

## 参考

[Java String](https://blog.csdn.net/justloveyou_/article/details/60983034)