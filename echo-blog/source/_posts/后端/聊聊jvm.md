---
title: 聊聊jvm
categories:
- 后端
tags: jvm
date: 
---
> jvm 是Java Virtual Machine（Java虚拟机）的缩写，java 虚拟机作为一种跨平台的软件是作用于操作系统之上的，那么认识并了解它的底层运行逻辑对于java开发人员来说很有必要！

让我们来看看它一次编译，到处运行的牛叉之处！
废话不多说，先看看jvm的架构图(无论何时脑子里要有这样一张图)：
![](https://s1.ax1x.com/2020/03/13/8nLiNj.png)

>总概  
**从这副架构图可以看出jvm由类装载器、运行时数据区、执行引擎、本地方法接口还有垃圾回收器构成;其中垃圾回收器作用在整个jvm内存中（主要作用在堆和方法区），所以图中没有具体体现，但是我们要知道这么个东西，后面会聊jvm内存调优主要是在调堆**

接下来，我们再结合jvm架构图一个个分析下：
### 「类加载器
了解类加载器就是了解代码编译执行的机制

1.加载：查找并加载类的二进制数据

2.连接：
- 验证：保证被加载的类的正确性；
- 准备：给类静态变量分配内存空间，赋值一个默认的初始值；
- 解析：把类中的符号引用转换为直接引用

  把java编译为class文件的时候，虚拟机并不知道所引用的地址；助记符：符号引用转为真正的直接引用，找到对应的直接地址！

3.初始化：给类的静态变量赋值正确的值；

来看下这行代码的执行顺序：
```java
package jvm;

public class TestClassLoader {
    public static int a = 1;
    // 1、加载  编译TestClassLoader文件为 .class 文件，通过类加载，加载到JVM

    // 2、连接
    //验证(1)  保证Class类文件没有问题
    //准备(2)  给int类型分配内存空间，a = 0；
    //解析(3)  符号引用转换为直接引用

    // 3、初始化
    //经过这个阶段的解析，把1 赋值给 变量 a；
}
```
了解了这个加载执行顺序后，我们再来看看下面这段代码，请你说说程序是如何输出的?

Demo1代码：
```java
package jvm;
/** 
*JVM 参数：
*-XX:+TraceClassLoading // 用于追踪类的加载信息并打印出来
*分析项目启动为什么这么慢，快速定位自己的类有没有被加载！
*rt.jar jdk 出厂自带的，最高级别的类加载器要加载的！
*/ 
public class Demo1 {
    public static void main(String[] args) {
        System.out.println(Children1.str2);
        Children1.sayHello();
        // 输出:
        //      Parent1 static
        //      Children1 static
        //      hello,str2
        //      hello,str1
    }
}
class Parent1{
    public static String str1 = "hello,str1";
    public static void sayHello(){
        System.out.println(str1);
    }

    static {
        System.out.println("Parent1 static");
    }
}
class Children1 extends Parent1{
    public static String str2 = "hello,str2";
    public static void sayHello(){
        System.out.println(str1);
    }
    
    static {
        System.out.println("Children1 static");
    }
}
```
可以看出，子类继承父类 会优先加载父类的 然后静态块是先于方法和静态变量的.那么常量又是怎样的呢？

Demo2代码：
```java
package jvm;

public class Demo2 {
    public static void main(String[] args) {
        System.out.println(Parent2.STR2);
        // 输出：
        // hello static final
    }
}
class Parent2{
    public static final String STR2 = "hello static final";
    static {
        System.out.println("Parent2 static"); // 这句话会输出吗？
    }
}
```
Demo3代码:
```java
package jvm;

import java.util.UUID;
public class Demo3 {
    public static void main(String[] args) {
        System.out.println(Parent3.STR3);
        // 输出： Parent3 static
        //       ee33c452-70e2-47a6-946a-99261819a49d
    }
}
class Parent3{
    public static final String STR3 = UUID.randomUUID().toString();
    static {
        System.out.println("Parent3 static"); // 这句话会输出吗？
    }
}
```
理解：根据上面的jvm架构图可以知道类信息，常量，静态变量，编译后运行代码都是存在方法区里的，而常量是存在方法区的常量池中的。

而对于编译期可以*“确定值”*的常量：例如Demo2中STR2 是存放在Demo2类调用者所在的常量池中的，当STR2放到常量池后Demo2与Parent2类的关系就没有了！

对于编译期*“不确定值”*的常量：例如Demo3中STR3 是不存在Demo3的调用者的常量池中的，在程序运行期间会主动使用Demo3所在的类，会加载其静态块！

上面的程序Parent2.STR2输出后就退出了Parent2类，因为程序加载Parent2类是直接调用了常量池中关于STR2的引用，无需再加载类的其他信息包括静态块，说白了按我的理解就是常量池和类的其他信息不在一个内存区，程序根据指令扫描的时候无需扫描不相关的内存区！说到池(池？嗯，等等是不是和线程池和队列有关，哈哈。学东西要养成发散思维的习惯，这样就能串起来很多知识点，渐渐形成自己的知识面。)接下来就来扩展一下：
```java
package jvm;
/**
 * 思考：
 * String == 比较的是什么？
 * 八大基本类型哪些实现了池化技术
 */
public class Exclude {
    public static void main(String[] args) {
        String java = "java";
        String java1 = new String("java");
        String java2 = new String("java");
        System.out.println(java == java1);//输出false 
        System.out.println(java1 == java2);//输出false
        // 注⚠️：String用==比较的时候不仅比较的是对象的值还比较的是对象值的引用
        // equals

        int i = 128;
        int i1 = 128;
        System.out.println(i == i1);//输出true

        Integer integer = 128;
        Integer integer1 = 128;
        System.out.println(integer == integer1);//输出false

        Float f = 0.1f;
        Float f1 = 0.1f;
        System.out.println(f == f1);//输出false
    }
}
```

注⚠️：java中基本类型的包装类的大部分都实现了常量池技术，这些类是Byte,Short,Integer,Long,Character,Boolean
另外两种浮点数类型的包装类Double Float则没有实现。
另外Byte,Short,Integer,Long,Character这5种整型的包装类也只是在对应值小于等于127时才可使用常量池，即对象不负责创建和管理大于127的这些类的对象

#### 类加载器 分类

1、java虚拟机自带的加载器

- BootStrap  根加载器 （加载系统的包，JDK 核心库中的类  rt.jar）
- Ext        扩展类加载器 （加载一些扩展jar包中的类）
- Sys/App    系统（应用类）加载器 （我们自己编写的类）

2、用户自己定义的加载器

- ClassLoader，只需要继承这个抽象类即可，自定义自己的类加载器

Demo4代码：
```java
package jvm;

public class Demo4 {
    public static void main(String[] args) {
        Object o = new Object(); // jdk 自带的
        Demo demo = new Demo();  // 实例化一个自己定义的对象

        // null 在这里并不代表没有，只是Java触及不到！
        System.out.println(o.getClass().getClassLoader()); // null
        System.out.println(demo.getClass().getClassLoader()); // AppClassLoader
        System.out.println(demo.getClass().getClassLoader().getParent()); // ExtClassLoader
        System.out.println(demo.getClass().getClassLoader().getParent().getParent()); // null
        
        // jvm 中有机制可以保护自己的安全；
        // 双亲委派机制 ： 一层一层的让父类去加载，如果顶层的加载器不能加载，然后再向下类推
        // ClassLoader         04
        // AppClassLoader      03
        // ExtClassLoader      02
        // BootStrap (最顶层)   01  java.lang.String  rt.jar

        // 双亲委派机制 可以保护java的核心类不会被自己定义的类所替代
    }
}
class Demo{

}
```
### 「本地接口库
>Native方法  **JNI ： Java Native Interface （Java 本地方法接口）**

大家都知道java是从C过度过来的，C可以直接操作硬件
思考一个问题：线程是基于内存的，那么应该是操作硬件 让计算机cpu来划分时间片来“同时”执行多个任务，那么程序是怎么怎么达到底层的呢？java 能直接操作硬件吗？我们来看看线程是怎么启动的，看源码（面向源码编程）
```java
 public synchronized void start() {
        /**
         * This method is not invoked for the main method thread or "system"
         * group threads created/set up by the VM. Any new functionality added
         * to this method in the future may have to also be added to the VM.
         *
         * A zero status value corresponds to state "NEW".
         */
        if (threadStatus != 0)
            throw new IllegalThreadStateException();

        /* Notify the group that this thread is about to be started
         * so that it can be added to the group's list of threads
         * and the group's unstarted count can be decremented. */
        group.add(this);

        boolean started = false;
        try {
            start0();  //启动 调用下面的native修饰的启动方法
            started = true;
        } finally {
            try {
                if (!started) {
                    group.threadStartFailed(this);
                }
            } catch (Throwable ignore) {
                /* do nothing. If start0 threw a Throwable then
                  it will be passed up the call stack */
            }
        }
    }

    private native void start0();
```
native : 只要是带了这个关键字的，说明 java的作用范围达不到，只能去调用底层 C 语言的库！

这部分知识作为java开发只需要了解即可，如果想去研究也可以看看嵌入式C的相关东西（C语言->编译器—>汇编语言—>驱动->硬件）。
### 运行时数据区
#### 程序计数器

每个线程都有一个程序计数器，是线程私有的。

程序计数器就是一块十分小的内存空间；几乎可以不计

作用： 看做当前字节码执行的行号指示器
![字节码](https://s1.ax1x.com/2020/03/13/8u0w0e.png)

分支、循环、跳转、异常处理！都需要依赖于程序计数器来完成！

`bipush`  将 int、float、String、常量值推送值栈顶

`istore` 将一个数值从操作数栈存储到局部变量表

`iadd ` 加

`imul` 乘
#### 方法区
Method Area 方法区 是 Java虚拟机规范中定义的运行是数据区域之一，和堆（heap）一样可以在线程之间共享！

**JDK1.7之前**

永久代：用于存储一些虚拟机加载类信息，常量，字符串、静态变量等等，这些东西都会放到永久代中；

永久代大小空间是有限的：如果满了 `OutOfMemoryError：PermGen `

**JDK1.8之后**

彻底将永久代移除  HotSpot jvm ，Java Heap 中或者 Metaspcace（Native Heap）元空间；

元空间就是方法区在   HotSpot jvm  的实现；

方法区主要是存：类信息，常量，字符串、静态变量、符号引用、方法代码。

元空间和永久代，都是对JVM规范中方法区的实现。

元空间和永久代最大的区别：元空间并不在Java虚拟机中，使用的是本地内存！

设置元空间大小：`-XX:MetasapceSize10m`
#### 堆和栈
堆和栈都是内存中相对独立的一块区域，jvm内存分为5个区域，分别是寄存器、本地方法区、方法区、栈内存、堆内存！

注⚠️：每个线程启动的时候，都会创建一个PC（Program Counter，程序计数器）寄存器，我们习惯上称呼为程序计数器，并不是什么其他的东西！

栈和堆的区别

- 1.堆是不连续的，所以分配的内存是在运行期确认的，因此大小不固定；堆对于整个应用程序都是共享、可见的；堆内存存储的是实体；堆内存存放的实体会被垃圾回收机制不定时的回收

- 2.栈是连续的，所以分配的内存大小要在编译期就确认，大小是固定的；栈只对于线程是可见的，所以也是线程私有，他的生命周期和线程相同（所以说，栈里面是一定不会存在垃圾回收的问题）；栈内存存储一些基本类型的值，对象的引用，方法等；栈内存的更新速度要快于堆内存（仅次于寄存器），因为局部变量的生命周期很短；栈内存存放的变量生命周期一旦结束就会被释放。

面试题：java中的基本数据类型都是存储在栈中的吗？

面试基本上问到这里都是三连，说说八大基本数据类型？说说堆和栈？接下来就是这个有坑的问题

答：不是

一：在方法中声明的变量，即该变量是局部变量，每当程序调用方法时，系统都会为该方法建立一个方法栈，其所在方法中声明的变量就放在方法栈中，当方法结束系统会释放方法栈，其对应在该方法中声明的变量随着栈的销毁而结束，这就局部变量只能在方法中有效的原因

在方法中声明的变量可以是基本类型的变量，也可以是引用类型的变量。

    （1）当声明是基本类型的对象时，其变量及变量的引用都在栈中。

    （2）当声明的是引用类型(new出来的)时，所声明的变量（该变量实际上是在方法中存储的是内存地址值）是放在方法的栈中，该变量所指向的对象是放在堆内存中的。

二：在类中声明的变量是成员变量，也叫全局变量，放在堆中的（因为全局变量不会随着某个方法执行结束而销毁）。

同样在类中声明的变量即可是基本类型的变量 也可是引用类型的变量

    （1）当为基本类型时，其变量名及其值放在堆内存中的

    （2）当为引用类型时，其声明的变量仍然会存储一个内存地址值，该内存地址值指向所引用的对象。引用变量名和对应的对象仍然存储在相应的堆中
扩展-->请根据以下代码说说6个对象的在堆栈中是如何分配的？
```java
String str1 = "abc";
String str2 = "abc";
String str3 = "abc";
String str4 = new String("abc");
String str5 = new String("333");
String str6 = new String("333");
```
结合下面的图来理解String 常量存放的位置：
![](https://s1.ax1x.com/2020/03/13/8uVR5n.png)

题外话：JVM目前有 SUN公司 HotSpot、BEA公司 JRockit、IBM公司 J9VM
关于堆在垃圾回收器里面介绍！
### 执行引擎
类装载器装载负责装载编译后的字节码，并加载到运行时数据区（Runtime Data Area），然后执行引擎执行会执行这些字节码。
字节码必须通过类加载过程加载到JVM后才能够被执行，执行有三种模式：
 - 1.解释执行
 - 2.JIT（Just-In-Time）编译即执行
 - 3.JIT与解释混合执行
来看下执行引擎的简要执行流程：

![JIT](https://s1.ax1x.com/2020/03/13/8u5DAI.png)

关于执行引擎这种底层的东西了解即可，说白了就是执行编译后字节码的一套概念模型！执行引擎包含了众多可以操作字节码的执行指令，使程序能够按照一定的顺序执行！关于执行引擎（推荐知乎<a href="https://zhuanlan.zhihu.com/p/43324258" target="_blank">层层剥开JVM——字节码执行引擎</a>,亦可以推荐孤尽老师《码出高效》一书中的第四章“走进JVM”来深入了解）

### 垃圾回收器
呼，长出了一口气！终于写到和堆相关的了
看堆之前我们先了解下JVM的内存布局：
![](https://s1.ax1x.com/2020/03/13/8uLBfs.png)
物理上只有 新生、养老；元空间在本地内存中，不在JVM中！

**GC 垃圾回收主要是在 新生区和养老区，又分为 普通的GC  和 Full GC**，如果堆满了，就会爆出 OutOfMemory(OOM);
> 新生区

新生区 就是一个类诞生、成长、消亡的地方！

新生区细分： Eden、s（from   to），所有的类Eden被 new 出来的，慢慢的当 Eden 满了，程序还需要创建对象的时候，就会触发一次轻量级GC；清理完一次垃圾之后，会将活下来的对象，会放入幸存者区（），....... 清理了 20次之后，出现了一些极其顽强的对象，有些对象突破了15次的垃圾回收！这时候就会将这个对象送入养老区！运行了几个月之后，养老区满了，就会触发一次 Full GC；假设项目1年后，整个空间彻彻底底的满了，突然有一天系统 OOM，排除OOM问题，或者重启；

Sun HotSpot 虚拟机中，内存管理（分代管理机制：不同的区域使用不同的算法！）

Eden from to

99% 的对象在 Eden 都是临时对象；

> 养老区

15次都幸存下来的对象进入养老区，养老区满了之后，触发 Full GC

默认是15次，可以修改！

> 永久区（Perm）

放一些 JDK 自身携带的 Class、Interface的元数据；

几乎不会被垃圾回收的；

`OutOfMemoryError：PermGen`  在项目启动的时候永久代不够用了？加载大量的第三方包！

JDK1.6之前： 有永久代、常量池在方法区；

JDK1.7：有永久代、但是开始尝试去永久代，常量池在堆中；

JDK1.8 之后：永久代没有了，取而代之的是元空间；常量池在元空间中；

闲聊：方法区和堆一样，是共享的区域，是JVM 规范中的一个逻辑的部分，但是记住它的别名 `非堆`

#### jvm内存调优
来看以下代码：
```java
package jvm;

/**
 * 环境:jdk1.8
 * jvm:HotSpot
 *
 *  默认情况：
 *  maxMemory : 1808.0MB （虚拟机试图使用的最大的内存量  一般是物理内存的 1/4）
 *  totalMemory : 123.0MB （虚拟机试图默认的内存总量 一般是物理内存的 1/64）
 *
 *  自定义： -XX:+PrintGCDetails; // 输出详细的垃圾回收信息
 *          -Xmx: 最大分配内存； 1/4
 *          -Xms: 初始分配的内存大小； 1/64
 *          -Xmx1024m -Xms1024m -XX:+PrintGCDetails
 */
public class HeapDemo {
    public static void main(String[] args) {
        // 获取堆内存的初始大小和最大大小
        long maxMemory = Runtime.getRuntime().maxMemory();
        long totalMemory = Runtime.getRuntime().totalMemory();

        System.out.println("maxMemory="+maxMemory+"(字节)、"+(maxMemory/1024/(double)1024)+"MB");
        System.out.println("totalMemory="+totalMemory+"(字节)、"+(totalMemory/1024/(double)1024)+"MB");
    }
}
```
输出信息如下：
![heapSpace](https://s1.ax1x.com/2020/03/13/8uvt3j.png)
由计算得出(305664K+699392K)/1024 = 981.5MB
说明新生代和老年代内存相加为整个分配内存大小，也从侧面证明了元空间不存在于jvm中，它存在于本地空间中！

--认识了堆后，我们再来聊聊堆的内存调优，这是每个java开发都需要掌握的！
模拟oom,看看GC详情：
```java
package jvm;

import java.util.Random;
/**
 * 设置内存大小：-Xmx8m -Xms8m -XX:+PrintGCDetails
 */
public class HeapDemo1 {
    public static void main(String[] args) {
        System.gc(); // 手动唤醒GC（），等待cpu的调用
        String str = "talk is cheap,show me your code";
        while (true){
            str += str
                    + new Random().nextInt(999999999)
                    + new Random().nextInt(999999999);
        }
        // 出现问题：java.lang.OutOfMemoryError: Java heap space
    }
}
```
输出如下：
```java
"C:\Program Files\Java\jdk1.8.0_171\bin\java.exe" -Xmx8m -Xms8m -XX:+PrintGCDetails "-javaagent:D:\idea\IntelliJ IDEA 2019.3.3\lib\idea_rt.jar=63475:D:\idea\IntelliJ IDEA 2019.3.3\bin" -Dfile.encoding=UTF-8 -classpath "C:\Program Files\Java\jdk1.8.0_171\jre\lib\charsets.jar;C:\Program Files\Java\jdk1.8.0_171\jre\lib\deploy.jar;C:\Program Files\Java\jdk1.8.0_171\jre\lib\ext\access-bridge-64.jar;C:\Program Files\Java\jdk1.8.0_171\jre\lib\ext\cldrdata.jar;C:\Program Files\Java\jdk1.8.0_171\jre\lib\ext\dnsns.jar;C:\Program Files\Java\jdk1.8.0_171\jre\lib\ext\jaccess.jar;C:\Program Files\Java\jdk1.8.0_171\jre\lib\ext\jfxrt.jar;C:\Program Files\Java\jdk1.8.0_171\jre\lib\ext\localedata.jar;C:\Program Files\Java\jdk1.8.0_171\jre\lib\ext\nashorn.jar;C:\Program Files\Java\jdk1.8.0_171\jre\lib\ext\sunec.jar;C:\Program Files\Java\jdk1.8.0_171\jre\lib\ext\sunjce_provider.jar;C:\Program Files\Java\jdk1.8.0_171\jre\lib\ext\sunmscapi.jar;C:\Program Files\Java\jdk1.8.0_171\jre\lib\ext\sunpkcs11.jar;C:\Program Files\Java\jdk1.8.0_171\jre\lib\ext\zipfs.jar;C:\Program Files\Java\jdk1.8.0_171\jre\lib\javaws.jar;C:\Program Files\Java\jdk1.8.0_171\jre\lib\jce.jar;C:\Program Files\Java\jdk1.8.0_171\jre\lib\jfr.jar;C:\Program Files\Java\jdk1.8.0_171\jre\lib\jfxswt.jar;C:\Program Files\Java\jdk1.8.0_171\jre\lib\jsse.jar;C:\Program Files\Java\jdk1.8.0_171\jre\lib\management-agent.jar;C:\Program Files\Java\jdk1.8.0_171\jre\lib\plugin.jar;C:\Program Files\Java\jdk1.8.0_171\jre\lib\resources.jar;C:\Program Files\Java\jdk1.8.0_171\jre\lib\rt.jar;E:\sCloud\studyDemo\out\production\studyDemo" jvm.HeapDemo1
[GC (Allocation Failure) [PSYoungGen: 1536K->504K(2048K)] 1536K->632K(7680K), 0.0022005 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (System.gc()) [PSYoungGen: 877K->488K(2048K)] 1005K->712K(7680K), 0.0017541 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (System.gc()) [PSYoungGen: 488K->0K(2048K)] [ParOldGen: 224K->645K(5632K)] 712K->645K(7680K), [Metaspace: 3441K->3441K(1056768K)], 0.0181221 secs] [Times: user=0.03 sys=0.00, real=0.02 secs] 
[GC (Allocation Failure) [PSYoungGen: 1249K->325K(2048K)] 1894K->971K(7680K), 0.0010536 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [PSYoungGen: 1531K->259K(2048K)] 2960K->1689K(7680K), 0.0009587 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [PSYoungGen: 1532K->487K(2048K)] 2961K->2165K(7680K), 0.0117368 secs] [Times: user=0.02 sys=0.00, real=0.01 secs] 
[GC (Allocation Failure) [PSYoungGen: 1688K->128K(2048K)] 6500K->4940K(7680K), 0.1107265 secs] [Times: user=0.02 sys=0.00, real=0.11 secs] 
[GC (Allocation Failure) [PSYoungGen: 128K->128K(2048K)] 4940K->4940K(7680K), 0.0211234 secs] [Times: user=0.00 sys=0.00, real=0.02 secs] 
[Full GC (Allocation Failure) [PSYoungGen: 128K->0K(2048K)] [ParOldGen: 4812K->2550K(5632K)] 4940K->2550K(7680K), [Metaspace: 3939K->3939K(1056768K)], 0.0311921 secs] [Times: user=0.03 sys=0.00, real=0.03 secs] 
[GC (Allocation Failure) [PSYoungGen: 138K->32K(2048K)] 4256K->4149K(7680K), 0.0064069 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 
[GC (Allocation Failure) [PSYoungGen: 32K->32K(2048K)] 4149K->4149K(7680K), 0.0007456 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (Allocation Failure) [PSYoungGen: 32K->0K(2048K)] [ParOldGen: 4117K->3332K(5632K)] 4149K->3332K(7680K), [Metaspace: 3942K->3942K(1056768K)], 0.0250859 secs] [Times: user=0.01 sys=0.00, real=0.02 secs] 
[GC (Allocation Failure) [PSYoungGen: 0K->0K(2048K)] 3332K->3332K(7680K), 0.0014623 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (Allocation Failure) [PSYoungGen: 0K->0K(2048K)] [ParOldGen: 3332K->3277K(5632K)] 3332K->3277K(7680K), [Metaspace: 3942K->3942K(1056768K)], 0.0241555 secs] [Times: user=0.02 sys=0.00, real=0.02 secs] 
Heap
 PSYoungGen      total 2048K, used 42K [0x00000000ffd80000, 0x0000000100000000, 0x0000000100000000)
  eden space 1536K, 2% used [0x00000000ffd80000,0x00000000ffd8a978,0x00000000fff00000)
  from space 512K, 0% used [0x00000000fff80000,0x00000000fff80000,0x0000000100000000)
  to   space 512K, 0% used [0x00000000fff00000,0x00000000fff00000,0x00000000fff80000)
 ParOldGen       total 5632K, used 3277K [0x00000000ff800000, 0x00000000ffd80000, 0x00000000ffd80000)
  object space 5632K, 58% used [0x00000000ff800000,0x00000000ffb337e8,0x00000000ffd80000)
 Metaspace       used 3974K, capacity 4568K, committed 4864K, reserved 1056768K
  class space    used 437K, capacity 460K, committed 512K, reserved 1048576K
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at java.util.Arrays.copyOf(Arrays.java:3332)
	at java.lang.AbstractStringBuilder.ensureCapacityInternal(AbstractStringBuilder.java:124)
	at java.lang.AbstractStringBuilder.append(AbstractStringBuilder.java:674)
	at java.lang.StringBuilder.append(StringBuilder.java:208)
	at jvm.HeapDemo1.main(HeapDemo1.java:11)

Process finished with exit code 1
// 分析：GC 普通GC(轻GC)、Full GC 重GC
// 1536K 执行 GC之前的大小,504K  执行 GC之后的大小
// (2048K) young 的total大小 ,(5632K)old 的total 大小 (7680K)整个堆已被占用的大小
// 0.0241555 secs 清理的时间、user 总计GC所占用CPU的时间   
// sys OS调用等待的时间   real 应用暂停的时间
```
好了，理解这些参数后基本知道怎么调优了。
其实Idea 还有一款插件`JProfiler` 专门用来分析程序内存占用情况的，用`JProfiler` 可以dump 内存快照帮助我们快速定位问题。教程百度上有，此处不做说明！

注⚠️：JVM在进行GC时，并非每次都是对三个区域进行扫描的！大部分的时候都是指的新生代！
- 普通GC：只针对新生代  【GC】
- 全局GC：主要是针对老年代，偶尔伴随新生代！ 【Full GC】

#### GC四大算法：
>引用计数法

特点：每个对象都有一个引用计数器，每当对象被引用一次，计数器就+1，如果引用失效，则计数器-1，如果为0，则GC可以清理；

缺点：

- 计数器维护麻烦！
- 循环引用无法处理！

注⚠️：JVM 一般不采用这种方式，现在一般使用可达性算法，GC Root...
>复制算法

1、一般普通GC 之后，差不多Eden几乎都是空的了！

2、每次存活的对象，都会被从 from 区和 Eden区等复制到 to区，from 和 to 会发生一次交换；记住一个点就好，**谁空谁是to**，每当幸存一次，就会导致这个对象的年龄+1；如果这个年龄值大于15（默认值，后面我们会讲解调整），就会进入养老区；

优点：没有标记和清除的过程！效率高！没有内存碎片！

缺点：需要浪费双倍的空间

Eden 区,对象存活率极低！ 统计：99% 对象都会在使用一次之后，引用失效！推荐使用 **复制算法**
>标记清除算法

老年代一般使用这个，但是会和我们后面的整理压缩一起使用！

优点：不需要额外的空间！

缺点：两次扫描，耗时较为严重，会产生内存碎片，不连续！

>标记清除压缩

减少了上面标记清除的缺点：没有内存碎片！但是耗时可能也较为严重！

那我们什么时候可以考虑使用这个算法呢？

在我们这个要使用算法的空间中，假设这个空间中很少，不经常发生GC，那么可以考虑使用这个算法！
```
总结：
内存效率：复制算法 > 标记清除算法 > 标记整理（时间复杂度！）
内存整齐度：复制算法=标记整理>标记清除算法
内存利用率：标记整理 = 标记清除算法 > 复制算法
```
#### 面试题：
最后再来看看几个面试题：

1.JVM 垃圾回收的时候如何确定垃圾，GC Roots？
- 引用计数法 （每引用一次就加一，为0或-1就会被GC清除）
- 可达性分析算法 (如果从 GC Root 这个对象开始，对于没有关联的对象GC认为不可达的，就会被清理)

2.什么是GC Root?
- 1、虚拟机栈中引用的对象！
- 2、类中静态属性引用的对象
- 3、方法区中的常量
- 4、本地方法栈中 Native 方法引用的对象！

3.jvm 常用参数有哪些，你都用过哪些？
- 1、标配参数：`-version`,`-help`,`-showversion`
- 2、X参数：`-Xint` # 解释执行，`-Xcomp` # 第一次使用就编译成本地的代码，`-Xmixed` # 混合模式（Java默认）
- 3、XX参数：+ 或者 - 某个属性值， + 代表开启某个功能，- 表示关闭了某个功能  例如:`jps -l` ,`jinfo -flag PrintGCDetail 进程号` # 查看某个运行中的java 程序，`-XX:+PrintGCDetails` # 打印GC详情
- 4.XX 参数之key = value值；例如：元空间大小：`-XX:MetaspaceSize=128m`#设置元空间大小，`-XX:MaxTenuringThreshold=15`#设置新生代需要经历多少次GC晋升到老年代中的最大阈值(默认值是15，最大也是15)

- 1).`-Xms` 初始堆的大小，等价： `-XX:InitialHeapSize`

- 2).`-Xmx` 最大堆的大小 ，等价：`-XX:MaxHeapSize`

` -XX:+PrintFlagsInitial` 查看 java 环境初始默认值；这里面只要显示的值，我们都可以手动赋值，不建议修改，了解即可！
![](https://s1.ax1x.com/2020/03/13/8KFXse.png)
`=` 默认值

`:=`  就是被修改过的值

`java -XX:+PrintCommandLineFlags -version`  打印出用户手动选项的 XX 选项

4.你常用的项目，发布后配置过JVM 调优参数吗？
```
-Xms
-Xmx
-Xss  : 线程栈大小设置，默认 512k~1024k
-Xmn  设置年轻代的大小，一般不用动！
-XX:MetaspsaceSize  设置元空间的大小，这个在本地内存中！
-XX:+PrintGCDetails
-XX:SurvivorRatio
设置新生代中的 s0/s1 空间的比例；
`uintx SurvivorRatio  = 8`  Eden：s0：s1 = 8:1:1
`uintx SurvivorRatio  = 4`  Eden：s0：s1 = 4:1:1
-XX:NewRatio
设置年轻代与老年代的占比：
`NewRatio  = 2`   新生代1，老年代是2，默认新生代整个堆的 1/3;
`NewRatio  = 4`   新生代1，老年代+是4，默认新生代整个堆的 1/5;
-XX:MaxTenuringThreshold
进入老年区的存活阈值；
`MaxTenuringThreshold  = 15`
```
>小结：至此JVM的知识，基本已总结完成，推荐书《码出高效》;买了很多书但是一直没看，我新增了栏目“拆书” 有时间会把买的书啃完了谢谢心得体会，或者看到精彩的地方会分享出来，当然不至于技术一类。为能看到这儿的自己点个赞吧，加油！
