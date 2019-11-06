---
layout: post
title: Java protected
categories: [Java Base]
tags: protected
---

# Java protected

类的访问权限：无（包访问权限），public（内部类除外）

类成员的访问权限：private,无，protected，public

package：一个项目中不可以有相同的两个包名，包括自定义的和引用的都不可以。

![](https://raw.githubusercontent.com/devin-jade/devin-imag/master/web/Java%E8%AE%BF%E9%97%AE%E6%9D%83%E9%99%90%E6%8E%A7%E5%88%B6.png)

其中包访问权限是java中默认的权限，具有包访问权限的类成员只能被同一个包里面的类访问；

## protected

继承访问权限，在默认包访问的基础上，让自己的子类看得见；

本质：

- 基类的protected成员是包内可见的，并对子类可见；
- 若子类和基类不再同一个包中，那么在子类里面，子列实例可以访问其从基类继承而来的protected方法，而不能访问基类实例的protected方法；

**在碰到涉及protected成员的调用时，首先要确定出该protected成员来自何方，其可见性范围是什么，然后就可以判断出当前用法是否可行了** 

- 看方法的最终来源是哪；
- 这个方法的的范围是本包以及子类；
- 看是不是同一个包，以及是不是访问的该子类继承下来的（该子类继承下来的包括该类自己，以及没有覆盖重写该变量的的子类）；

```java
//示例一
package p1;
public class Father1 {
    protected void f() {}    // 父类Father1中的protected方法
}

package p1;
public class Son1 extends Father1 {}

package p11;
public class Son11 extends Father1{}

package p1;
public class Test1 {
    public static void main(String[] args) {
        Son1 son1 = new Son1();
        son1.f(); // f()来源于Father，可见性是p1以及子类Son1，Son11，test在P1，Compile OK    
        son1.clone(); // clone来源于Object，可见性是java.long以及所有子类，Test1不是子类，也不再lang包 Compile Error
        Son11 son = new Son11();    
        son11.f(); //f()来源于Father，可见性是p1以及子类Son1，son11，test在P1，Compile OK
        son11.clone(); // clone来源于Object，可见性是java.long以及所有子类，Test1不是子类，也不再lang包 Compile Error
    }
}

//示例二
package p2;
class MyObject2 {
    protected Object clone() throws CloneNotSupportedException{
       return super.clone();
    }
}

package p22;
public class Test2 extends MyObject2 {
    public static void main(String args[]) {
       MyObject2 obj = new MyObject2();
       obj.clone(); // clone来源于MyObject2，可见范围是p2以及MyObject2的子类，Test2是子类，但是obj实际访问的MyObject2的clone，不是继承下来的；Compile Error         
       Test2 tobj = new Test2();
       tobj.clone(); // clone来源于MyObject2，可见范围是p2以及MyObject2的子类，Test2是子类，访问的Test2继承下来的clone，Complie OK
    }
}

//示例三
package p3;
class MyObject3 extends Test3 {
}

package p33;
public class Test3 {
  public static void main(String args[]) {
    MyObject3 obj = new MyObject3();
    obj.clone();   // clone是Object的，可见范围是java.long以及所有子类，Test3是子类，而且obj访问的是Test3继承下来的clone Compile OK  
  }
}

//示例四
package p4;
class MyObject4 extends Test4 {
  protected Object clone() throws CloneNotSupportedException {
    return super.clone();
  }
}

package p44;
public class Test4 {
  public static void main(String args[]) {
    MyObject4 obj = new MyObject4();
    obj.clone(); // clone来源于MyObject4，可见范围p4和所有子类，Test4，不是MyObject4的子类，Test4也不再lang Compile Error  
  }
}

//示例五
package p5;
class MyObject5 {
    protected Object clone() throws CloneNotSupportedException{
       return super.clone();
    }
}
public class Test5 {
    public static void main(String[] args) throws CloneNotSupportedException {
       MyObject5 obj = new MyObject5();
       obj.clone(); // clone 来源于MyObject5，可见范围是p5以及MyObject5的子类，Test5在p5，Compile OK 
    }
}

//示例六
package p6;
class MyObject6 extends Test6{}
public class Test6 {
  public static void main(String[] args) {
    MyObject6 obj = new MyObject6();
    obj.clone(); // Clone来源于Object，可见范围java.lang以及所有子类，Test6是子类，obj访问的是Test6继承下来的clone Compile OK 
  }
}

//示例七
package p7;
class MyObject7 extends Test7 {
    public static void main(String[] args) {
        Test7 test = new Test7();
        test.clone(); //clone来源于Object，可见范围java.lang以及所有子类，MyObject7是子类，但是test访问的不是MyObject7继承的clone Compile Error  
  }
}
public class Test7 {
}
```

## 其他修饰符

- static：修饰变量和内部类(不能修饰常规类)，其中所修饰变量称为类变量或静态变量。静态变量是和类层次的变量,每个实例共享这个静态变量，在类加载时初始化。
- final：被声明为final的变量必须在声明时给定初值（当然，空白final可以延迟到构造器中赋值），而且被修饰的变量不能修改值。当修饰类时，该类不能派生出子类；修饰方法时，该方法不能被子类覆盖。
- abstract：修饰类和方法。当修饰类时，该类不能创建对象；修饰方法时，为抽象方法。类只要有一个abstract方法，类就必须定义为abstract，但abstract类不一定非要有abstract方法不可。

## 参考

[protected](https://blog.csdn.net/justloveyou_/article/details/61672133)

