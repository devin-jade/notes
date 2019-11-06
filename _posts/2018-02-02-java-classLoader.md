---
layout: post
title: Java String
categories: [classLoader]
tags: java 基础知识
---

# Java ClassLoader

类加载器：**通过一个类的全限定名来获取描述此类的二进制字节流**功能的实现。类加载器在JVM之外，开发者可以自行决定如何去获取所需要的类。

每一个类加载器有自己独立的**类名称空间**。所以比较两个类是否相等，只有在两个类是由同一个类加载器加载的前提下才有意义。否则即使两个类是同一个JVM虚拟机，加载的同一份class文件，只有他们的类加载器不同，那么他们就是不同的类。

两个类相等是Class对象的equals方法，isAssignableFrom方法，isInstance方法返回的结果，也包含instanceof关键字的判断相等。

## 双亲委派模型

![](https://raw.githubusercontent.com/devin-jade/devin-imag/master/web/20191105223420.png)

### 双亲委派模型工作工程

如果一个类加载器收到了类加载的请求，它首先不会自己去尝试加载这个类，而是吧这个请求委派给父类加载器去完成，每一层的类加载器都是如此，所以所有的加载请求最终都应该传送到顶层的启动类加载器中，只有父加载器反馈自己无法完成这个加载请求时（在他的搜索范围内没有找到所需的类），子加载器才会尝试自己去加载。

### 双亲委派模型优势

对于需要使用单例模式的类很容易实现，因为会最终使用相同的加载器加载。

使用双亲委派模型来组织类加载器之间的关系，有一个显而易见的好处就是Java类随着它的类加载器一起具备了一种带有优先级的层次关系。 例如类java.lang.Object，它存放在rt.jar之中，无论哪一个类加载器要加载这个类，最终都是委派给处于模型最顶端的启动类加载器进行加载，因此Object类在程序的各种类加载器环境中都是同一个类。 相反，如果没有
使用双亲委派模型，由各个类加载器自行去加载的话，如果用户自己编写了一个称为java.lang.Object的类，并放在程序的ClassPath中，那系统中将会出现多个不同的Object类，Java类型体系中最基础的行为也就无法保证，应用程序也将会变得一片混乱。 如果读者有兴趣的话，可以尝试去编写一个与rt.jar类库中已有类重名的Java类，将会发现可以正常编
译，但永远无法被加载运行 。

Java 官方建议开发者加载类的时候遵循双亲委派模型。

```java
  protected Class<?> loadClass(String name, boolean resolve)
        throws ClassNotFoundException {
        synchronized (getClassLoadingLock(name)) {
            // First, check if the class has already been loaded
            Class<?> c = findLoadedClass(name);
            if (c == null) {
                long t0 = System.nanoTime();
                try {
                    if (parent != null) {
                        c = parent.loadClass(name, false);
                    } else {
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                    // 如果父类抛出了ClassNotFoundException 说明父加载器无法完成加载请求
                }
                if (c == null) {
                    // If still not found, then invoke findClass in order to find the class.
                    // 父类无法加载，调用子类的findClass进行类加载
                    long t1 = System.nanoTime();
                    c = findClass(name);

                    // this is the defining class loader; record the stats
                    sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                    sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                    sun.misc.PerfCounter.getFindClasses().increment();
                }
            }
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
    }
```



## 破坏双亲委派模型

双亲委派模型并不是一个强制性的约束模型，而是Java设计者推荐的模式，但是任何一个模式都有好坏，因此也可以打破这个规则。

### 线程上下文类加载器

#### Java SPI

Java SPI：service provider interface 为某个接口寻找服务实现的机制；在面向对象的设计中，我们推荐模块之间基于接口编程，模块之间不对实现类进行硬编码，做到可插拔。那么就需要一种机制将装配的控制权移到程序之外，即Java SPI模块。例子日志模块，xml解析模块，jdbc模块。

#### Java SPI 约定

当服务的提供者提供了服务接口的一种实现之后，在jar包的META-INF/services/目录里同时创建一个以服务接口命名的文件，该文件里就是实现该服务接口的具体实现类。而当外部程序装配这个模块的时候，就能通过该jar包META-INF/services/里的配置文件找到具体的实现类名，并装载实例化，完成模块的注入。基于这样一个约定就能很好的找到服务接口的实现类，而不需要再代码里制定。

> jdk提供的服务实现的查找工具类：java.util.ServiceLoader.

![](https://raw.githubusercontent.com/devin-jade/devin-imag/master/web/20191105232103.png)

#### SPI问题

SPI的接口是Java核心库的一部分，是由**启动类加载器**来加载的；而SPI的实现类是由**系统类加载器**来加载的。启动类加载器是无法找到 SPI 的实现类的(因为它只加载 Java 的核心库)，按照双亲委派模型，启动类加载器无法委派系统类加载器去加载类。也就是说，类加载器的双亲委派模式无法解决这个问题。

那怎么解决这个问题呢，Java设计者引入了一个不怎么优雅的设计，线程上下文类加载器。

> 线程上下文类加载器(Tread Context ClassLoader)：Java.lang.Thread中的方法 getContextClassLoader()和 setContextClassLoader(ClassLoader cl)用来获取和设置线程的上下文类加载器。如果没有通过 setContextClassLoader(ClassLoader cl)方法进行设置的话，线程将继承其父线程的上下文类加载器，如果在应用程序的全局范围内都没有设置过的话，那这个类加载器默认就是应用程序类加载器（系统类加载器）。

直白一点说就是：我（JDK）提供了一种帮你（第三方实现者）加载服务（如数据库驱动、日志库）的便捷方式，只要你遵循约定（把类名写在/META-INF里），那当我启动时我会去扫描所有jar包里符合约定的类名，再调用forName加载。但我的ClassLoader是没法加载的，那就把它加载到当前执行线程的线程上下文类加载器里，后续你想怎么操作就是你的事了。

### 代码热部署（HotSwap）模块热部署（HotDeployment）

  Java模块化标准 OSGI:  OSGi实现模块化热部署的关键则是它自定义的类加载器机制的实现。 每一个程序模块（OSGi中称为Bundle）都有一个自己的类加载器，当需要更换一个Bundle时，就把Bundle连同类加载器一起换掉以实现代码的热替换。  

在OSGi环境下，类加载器不再是双亲委派模型中的树状结构，而是进一步发展为更加复杂的网状结构，当收到类加载请求时，OSGi将按照下面的顺序进行类搜索：

1. 将以java.*开头的类委派给父类加载器加载。
2. 否则，将委派列表名单内的类委派给父类加载器加载。
3. 否则，将Import列表中的类委派给Export这个类的Bundle的类加载器加载。
4. 否则，查找当前Bundle的ClassPath，使用自己的类加载器加载。
5. 否则，查找类是否在自己的Fragment Bundle中，如果在，则委派给Fragment Bundle的类加载器加载。
6. 否则，查找Dynamic Import列表的Bundle，委派给对应Bundle的类加载器加载。
7. 否则，类查找失败。

上面的查找顺序中只有开头两点仍然符合双亲委派规则，其余的类查找都是在平级的类加载器中进行的。笔者虽然使用了“被破坏”这个词来形容上述不符合双亲委派模型原则的行为，但这里“被破坏”并不带有贬义的感情色彩。 只要有足够意义和理由，突破已有的原则就可认为是一种创新。 正如OSGi中的类加载器并不符合传统的双亲委派的类加载器，并且业界对其为了实现热部署而带来的额外的高复杂度还存在不少争议，但在Java程序员中基本有一个共识：OSGi中对类加载器的使用是很值得学习的，弄懂了OSGi的实现，就可以算是掌握了类加载器的精髓。  





## 参考

[双亲委派模型与线程上下文类加载器](https://blog.csdn.net/justloveyou_/article/details/72231425)