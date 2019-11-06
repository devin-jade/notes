---
layout: post
title: Java 常量池
categories: [Java]
tags: constant pool

---

# Java 常量池

java 常量池是线程共享的，一般包含字面量和符号引用。

 ![](https://raw.githubusercontent.com/devin-jade/devin-imag/master/web/20191103220716.png)

java 存在三种常量池和一种对象池，class文件的静态常量池，运行时的常量池，字符串的常量池，包装类的对象池。

- Class 文件静态常量池 是编译期生成的 Class 文件中的常量池
- 运行时常量池 是 Class 常量池 在运行时的表示形式
- 字符串常量池 是缓存字符串的，全局共享，它保存的是 String 实例对象的引用

## Class 文件的静态常量池

常量池中主要存放两大类常量：字面量（Literal） 和 符号引用（Symbolic Reference），字面量比较接近于 Java 语言层面的常量概念，如文本字符串 、声明为 final 的常量值等。而符号引用则属于编译原理方面的概念，包括了下面三类常量：

- 类和接口的全限定名（Fully Qualified Name）
- 字段的名称和描述符（Descriptor）
- 方法的名称和描述符

 静态常量池***\*用于存放编译期生成的各种字面量和符号引用，这部分内容将在类加载后存放到方法区的运行时常量池中。\****其中符号引用其实引用的就是常量池里面的字符串，但符号引用不是直接存储字符串，而是存储字符串在常量池里的索引。 

通过 javap 命令可以看到 Class 文件的常量池部分。

## 运行时常量池

运行时常量池是静态常量池运行时的表现，JVM中类的加载过程是，加载，连接，初始化。 将 .class 文件中的静态常量池转换为方法区的运行时常量池发生在“Loading”阶段，而符号引用替换为直接引用发生在 “Resolution”阶段。 

![](https://raw.githubusercontent.com/devin-jade/devin-imag/master/web/20191103220539.png)

 当Class文件被加载完成后，java虚拟机会将静态常量池里的内容转移到运行时常量池里，在静态常量池的符号引用有一部分是会被转变为直接引用的，比如说类的静态方法或私有方法，实例构造方法，父类方法，这是因为这些方法不能被重写其他版本，所以能在加载的时候就可以将符号引用转变为直接引用，而其他的一些方法是在这个方法被第一次调用的时候才会将符号引用转变为直接引用的。 

 运行时常量池（Runtime Constant Pool）是方法区的一部分，它是 Class 文件中每一个类或接口的常量池表的运行时表示形式。Class 常量池中存放的编译期生成的各种字面量和符号引用，将在类加载后进入方法区的运行时常量池中存放。 ，一般来说，除了保存Class文件中描述的符号引用外，还会把翻译出来的直接引用也存储在运行时常量池中。 

方法区与 Java 堆一样，是各个线程共享的内存区域，它用于存储已被虚拟机加载的类信息、常量、静态常量、即时编译器编译后的代码等数据。虽然 Java 虚拟机规范把方法区描述为堆的一个逻辑部分，但是它却有一个别名叫 Non-Heap（非堆）。目的应该是与 Java 堆区分开来。

## 字符串常量池

 字符串常量池是用来缓存字符串的。对于需要重复使用的字符串，每次都去 new 一个 String 实例，无疑是在浪费资源，降低效率。所以，JVM 一般会维护一个字符串常量池，它是全局共享的，你可是把它看成是一个 HashSet。需要注意的是，它保存的是堆中字符串实例的引用，并不存储实例本身。 

### **String.intern()**

JDK1.6：

 当调用 intern 方法时，如果池已经包含一个等于此 String 对象的字符串（用 equals(Object) 方法确定），则返回池中的字符串。否则，将此 String 对象添加到池中，并返回此 String 对象的引用。

![](https://raw.githubusercontent.com/devin-jade/devin-imag/master/web/JDK6.png)

JDK1.7之后：

  查找当前字符串常量池是否存在该字符串的引用，如果存在直接返回引用；如果不存在，则在堆中创建该字符串实例，并返回其引用。 

![](https://raw.githubusercontent.com/devin-jade/devin-imag/master/web/JDK8.png)

```java
String s1 = new String("he") + new String("llo");
String s2 = s1.intern();
 
System.out.println(s1 == s2);
// 在 JDK 1.6 下输出是 false，创建了 6 个对象
// 在 JDK 1.7 之后的版本输出是 true，创建了 5 个对象
// 当然我们这里没有考虑GC，但这些对象确实存在或存在过
```

## 包装类对象池

包装类的对象池(也有称常量池)和JVM的静态/运行时常量池没有任何关系。静态/运行时常量池有点类似于符号表的概念，与对象池相差甚远。包装类的对象池是池化技术的应用，并非是虚拟机层面的东西，而是 Java 在类封装里实现的。

Integer 源码：

```java
private static class IntegerCache {
        static final int low = -128;
        static final int high;
        static final Integer cache[];
 
        static {
// high value may be configured by property
            int h = 127;
            String integerCacheHighPropValue =
                    sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
            if (integerCacheHighPropValue != null) {
                try {
                    int i = parseInt(integerCacheHighPropValue);
                    i = Math.max(i, 127);
// Maximum array size is Integer.MAX_VALUE
                    h = Math.min(i, Integer.MAX_VALUE - (-low) - 1);
                } catch (NumberFormatException nfe) {
// If the property cannot be parsed into an int, ignore it.
                }
            }
            high = h;
 
            cache = new Integer[(high - low) + 1];
            int j = low;
            for (int k = 0; k < cache.length; k++)
                cache[k] = new Integer(j++);
 
// range [-128, 127] must be interned (JLS7 5.1.7)
            assert IntegerCache.high >= 127;
        }
 
        private IntegerCache() {
        }
    }
 
    public static Integer valueOf(int i) {
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }
```

IntegerCache 是 Integer 在内部维护的一个静态内部类，用于对象缓存。通过源码我们知道，Integer 对象池在底层实际上就是一个变量名为 cache 的数组，里面包含了 -128 ～ 127 的 Integer 对象实例。

　　使用对象池的方法就是通过 Integer.valueOf() 返回 cache 中的对象，像 Integer i = 10 这种自动装箱实际上也是调用 Integer.valueOf() 完成的。

　　如果使用的是 new 构造器，则会跳过 valueOf()，所以不会使用对象池中的实例。

```java
Integer i1 = 10;
Integer i2 = 10;
Integer i3 = new Integer(10);
Integer i4 = new Integer(10);
Integer i5 = Integer.valueOf(10);
 
System.out.println(i1 == i2); // true
System.out.println(i2 == i3); // false
System.out.println(i3 == i4); // false
System.out.println(i1 == i5); // true
```

 注意到注释中的一句话 “The cache is initialized on first usage”，缓存池的初始化在第一次使用的时候已经全部完成，这涉及到设计模式的一些应用。这和常量池中字面量的保存有很大区别，Integer 不需要显示地出现在代码中才添加到池中，初始化时它已经包含了所有需要缓存的对象。 

## 参考

[Java 8 中的常量池、字符串池、包装类对象池](https://www.cnblogs.com/itplay/p/11137526.html)

[JVM中三个常量池（两种常量池）的解析及其随jdk版本的变化](https://www.javatt.com/p/47643)