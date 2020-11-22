---
title: JMM 内存模型知识点探究
categories:
- 后端
tags:
date:
---
>Java  Memory  Model  Java内存模型；就是一个理论! 线程安全相关~！

### 八大操作：

内存交互操作有8种，虚拟机实现必须保证每一个操作都是原子的，不可再分的（对于double和long类型的变量来说，load、store、read和write操作在某些平台上允许例外）

* lock （锁定）：作用于主内存的变量，把一个变量标识为线程独占状态
* unlock（解锁）：作用于主内存的变量，它把一个处于锁定状态的变量释放出来，释放后的变量才可以被其他线程锁定
* read（读取）：作用于主内存变量，它把一个变量的值从主内存传输到线程的工作内存中，以便随后的load动作使用
* load   (载入）：作用于工作内存的变量，它把read操作从主存中变量放入工作内存中
* use （使用）：作用于工作内存中的变量，它把工作内存中的变量传输给执行引擎，每当虚拟机遇到一个需要使用到变量的值，就会使用到这个指令
* assign（赋值）：作用于工作内存中的变量，它把一个从执行引擎中接受到的值放入工作内存的变量副本中
* *store （存储）：作用于主内存中的变量，它把一个从工作内存中一个变量的值传送到主内存中，以便后续的write使用
* write（写入）：作用于主内存中的变量，它把store操作从工作内存中得到的变量的值放入主内存的变量中

**JMM对这八种指令的使用，制定了如下规则：**

- 不允许read和load、store和write操作之一单独出现。即使用了read必须load，使用了store必须write
- 不允许线程丢弃他最近的assign操作，即工作变量的数据改变了之后，必须告知主存 （可见）
- 不允许一个线程将没有assign的数据从工作内存同步回主内存
- 一个新的变量必须在主内存中诞生，不允许工作内存直接使用一个未被初始化的变量。就是怼变量实施use、store操作之前，必须经过assign和load操作
- 一个变量同一时间只有一个线程能对其进行lock。多次lock后，必须执行相同次数的unlock才能解锁
- 如果对一个变量进行lock操作，会清空所有工作内存中此变量的值，在执行引擎使用这个变量前，必须重新load或assign操作初始化变量的值
- 如果一个变量没有被lock，就不能对其进行unlock操作。也不能unlock一个被其他线程锁住的变量
- 对一个变量进行unlock操作之前，必须把此变量同步回主内存

### Volatile 关键字

>1、保证可见性 （JMM）
```java
package jmm;

import java.util.concurrent.TimeUnit;
public class demo1 {
    // private  static int num = 0;—>导致主线程修改后A 线程并没有感知到num的变化 从而不会退出循环
    private volatile static int num = 0;// 加了volition 关键字 保证了对象的可见性
    public static void main(String[] args) throws InterruptedException {
        new Thread(()->{
            while (num==0){ // 没有加 volatile 的时候，这个对象不可见

            }
        },("A")).start();

        TimeUnit.SECONDS.sleep(1);
        num = 1; // 虽然main线程修改了这个值，但是上面的线程并不知道！
        System.out.println("修改后 "+num);
    }
}
```
>2、不保证原子性 （核心难点：原子类）
```java
package jmm;

public class demo2 {
    private volatile static int num = 0;
    //不加synchronized 时不能保证其原子性
//    public  static void add(){
//        num++;
//    }
    public synchronized static void add(){
        num++;
    }
    public static void main(String[] args) {
        // 期望 num 最终是 2 万
        for (int i = 1; i <=20 ; i++) {
            new Thread(()->{
                for (int j = 1; j <= 1000; j++) {
                    add();
                }
            },String.valueOf(i)).start();
        }

        // 判断活着的线程
        while (Thread.activeCount()>2){ // mian  gc
            Thread.yield();
        }
        System.out.println(Thread.currentThread().getName() + " " + num);
    }
}
```
jmm底层逻辑
![](https://s2.ax1x.com/2020/03/11/8AutDs.png)

### 扩展
----那么，请你说说，如果不用 synchronized 和 lock ，如何解决这个问题？
这个时候人家问的是AtomicInteger
```java
package jmm;

import java.util.concurrent.atomic.AtomicInteger;
public class demo3 {
    // int 不是原子性的
    // AtomicInteger底层用的是 volatile 关键字 AtomicInteger
    private static AtomicInteger num = new AtomicInteger();
    // synchronized
    public static void add(){
        //num++; // 不是一个原子性操作
        num.getAndIncrement(); // num++
        // Unsafe 类：
        // Java不能直接操作内存！  native  c++=> 操作内存
        // Unsafe 后门，可以通过它直接操作内存！
    }
    public static void main(String[] args) {
        // 期望 num 最终是 2 万
        for (int i = 1; i <=20 ; i++) {
            new Thread(()->{
                for (int j = 1; j <= 1000; j++) {
                    add();
                }
            },String.valueOf(i)).start();
        }
        // 判断活着的线程
        while (Thread.activeCount()>2){ // mian  gc
            Thread.yield();
        }

        System.out.println(Thread.currentThread().getName() + " " + num);
    }
}
```
AtomicInteger 底层实现用了volatile 关键字和Unsafe类之间操作内存
![](https://s2.ax1x.com/2020/03/11/8AMilD.png)

----AtomicInteger 扩展到cas(比较并交换)
```java
package jmm;

import java.util.concurrent.atomic.AtomicInteger;
public class casDemo {
    public static void main(String[] args) {
        // AtomicInteger 默认为 0
        AtomicInteger atomicInteger = new AtomicInteger(5);

        // compareAndSet   CAS 比较并交换
        // public final boolean compareAndSet(int expect, int update)
        // 如果这个值是期望的值，那么则更新为指定的值
        System.out.println(atomicInteger.compareAndSet(5, 20));

        System.out.println(atomicInteger.get());
        // 如果这个值是期望的值，那么则更新为指定的值
        System.out.println(atomicInteger.compareAndSet(20, 6));

        System.out.println(atomicInteger.get());
    }
}
```
`getAndIncrement` 实现了 int ++的操作！
```java
getAndIncrement() ;
// unsafe可以直接操作内存
public final int getAndIncrement() {
    // this 调用的对象
    // valueOffset 当前这个对象的值的内存地址偏移值
    // 1
    return unsafe.getAndAddInt(this, valueOffset, 1);
}

public final int getAndAddInt(Object var1, long var2, int var4) {
    int var5;
    do { // 自旋锁（就是一直判断！）
        // var5 = 获得当前对象的内存地址中的值！
        var5 = this.getIntVolatile(this, valueOffset);
        // compareAndSwapInt 比较并交换
        // 比较当前的值 var1 对象的var2地址中的值是不是 var5，如果是则更新为 var5 + 1
        // 如果是期望的值，就交换，否则就不交换！
    } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

    return var5;
}
```
**CAS缺点：**

1、循环开销很大！

2、内存操作，每次只能保证一个共享变量的原子性！

3、出现ABA 问题？

>3.禁止指令重排 （核心难点：说出单例模式。说出CAS。说出CPU原语）

* 单例模式：懒汉式 饿汉式（设计模式中）
* CAS：比较并交换
* CPU原语：原语，一般是指由若干条指令组成的程序段，用来实现某个特定功能，在执行过程中不可被中断。
原语是操作系统核心（不是由进程，而是由一组程序模块组成）的一个组成部分，并且常驻内存，通常在管态下执行。原语一旦开始执行，就要连续执行完，不允许中断

指令重排：就是你的写程序不一定是按照你的程序跑的？

源代码->编译器（优化重排）->指令并行重排-> 内存系统的重排-> 最终执行的！

***单线程一定安全！（但是，也不能避免指令重排！）***

处理器在进行重排的时候会==考虑指令之间的依赖性！==

理解多线程下的指令重排问题：
```
int x,y,a,b = 0;
线程1                       线程2
x = a;                     y = b;
b = 1;                     a = 2;
理想的结果： x=0  y = 0

指令重排：
线程1                       线程2
b = 1;                     a = 2;
x = a;                     y = b;

重排后的结果： x=2  y = 1
```
votatile 可以禁止指令重排！

内存屏障（Memory  Barrier）：CPU的指令；两个作用：

1、保证特定的执行顺序！

2、保证某些变量的内存可见性 （votatile就是用它这个特性来实现的）

如图：
![](https://s2.ax1x.com/2020/03/11/8ANdaD.png)
**请你谈谈指令重排的最经典的应用！DCL单例模式**

推荐一篇分析DCL单例模式的<a href="https://www.iteye.com/blog/wjy320-2052991" target="_blank">博客</a>

### ABA 问题
> 原子类来解决（通过原子引用）

通过增加一个版本号来解决，和乐观锁一模一样！
```java
package jmm;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicStampedReference;
/**
 * 经典aba 问题
 */
public class abaDemo {
    // version = 1
    static AtomicStampedReference<Integer> atomicReference = new AtomicStampedReference<>(100,1);

    public static void main(String[] args) {
        // 其他人员 小花，需要每次执行完毕 + 1
        new Thread(()->{
            int stamp = atomicReference.getStamp();// 获得版本号
            System.out.println("T1 stamp01=>"+stamp);
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            atomicReference.compareAndSet(100,101,
                    atomicReference.getStamp(),atomicReference.getStamp()+1);
            System.out.println("T1 stamp02=>"+atomicReference.getStamp());

            atomicReference.compareAndSet(101,100,
                    atomicReference.getStamp(),atomicReference.getStamp()+1);
            System.out.println("T1 stamp03=>"+atomicReference.getStamp());
        },"T1").start();


        // 乐观的小明
        new Thread(()->{
            int stamp = atomicReference.getStamp();// 获得版本号
            System.out.println("T2 stamp01=>"+stamp);
            try {
                TimeUnit.SECONDS.sleep(3);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            boolean result = atomicReference.compareAndSet(100, 1, stamp, stamp + 1);
            System.out.println("T2 是否修改成功："+ result);
            System.out.println("T2 stamp02=>"+atomicReference.getStamp());
            System.out.println("T2 当前获取得最新的值=>"+atomicReference.getReference());
        },"T2").start();
    }
}
```

### 探究锁

1.自旋锁
上面列举了unsafe 类的源码 getAndAddInt
自己写一个自旋锁：
Lock类：
```java
package jmm;

import java.util.concurrent.atomic.AtomicReference;
public class Lock {
    // 锁线程
    // AtomicInteger 默认 是 0
    // AtomicReference 默认是 null
    AtomicReference<Thread> atomicReference = new AtomicReference<>();

    public void lock(){
        Thread thread = Thread.currentThread();
        System.out.println(thread.getName()+"===> lock");
        // 上锁  自旋
        while (!atomicReference.compareAndSet(null,thread)){

        }
    }
    // 解锁
    public void unlock() {
        Thread thread = Thread.currentThread();
        atomicReference.compareAndSet(thread, null);
        System.out.println(thread.getName() + "===> unlock");
    }
}
```
测试类：
```java
package jmm;

import java.util.concurrent.TimeUnit;
/**
*T1===> lock
*T2===> lock
*T1===> unlock
*T2===> unlock
*/
public class lockTest {
    public static void main(String[] args) {
        Lock lock = new Lock();

        // 1 一定先拿到锁
        new Thread(()->{
            lock.lock();
            try {
                TimeUnit.SECONDS.sleep(5);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            lock.unlock();
        },"T1").start();

        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        // 2
        new Thread(()->{
            lock.lock();
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            lock.unlock();
        },"T2").start();
    }
}
```

2.死锁，死锁的排查
什么是死锁？
死锁是指两个或两个以上的线程在执行过程中，由于竞争资源或者由于彼此通信而造成的一种阻塞的现象。

![](https://s2.ax1x.com/2020/03/11/8ArQ3D.png)

示例代码：
```java
package jmm;

import java.util.concurrent.TimeUnit;
public class MyLockThread implements  Runnable {
    private String lockA;
    private String lockB;

    public MyLockThread(String lockA, String lockB) {
        this.lockA = lockA;
        this.lockB = lockB;
    }

    @Override
    public void run() {
        synchronized (lockA){
            System.out.println(Thread.currentThread().getName()+"lock:"+lockA+"=>get:"+lockB);
            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            synchronized (lockB){
                System.out.println(Thread.currentThread().getName()+"lock:"+lockB+"=>get:"+lockA);
            }
        }
    }
}

```
测试代码：
```java
package jmm;
// 面对死锁你该怎么办？
// 日志
// 查看堆栈信息！ jmm 的知识
// 1、获取当前运行的java进程号   jps -l
// 2、查看信息                 jstack 进程号
// 3、jconsole 查看对应的信息！(可视化工具！)
// ......
public class DeadLockTest {
    public static void main(String[] args) {
        String lockA = "lockA";
        String lockB = "lockB";

        new Thread(new MyLockThread(lockA,lockB),"T1").start();
        new Thread(new MyLockThread(lockB,lockA),"T2").start();
    }
}

```
>小结：jmm是jvm的一种规范，定义了jvm的内存模型。
它屏蔽了各种硬件和操作系统的访问差异，不像c那样直接访问硬件内存，相对安全很多。
它的主要目的是解决由于多线程通过共享内存进行通信时，存在的本地内存数据不一致、
编译器会对代码指令重排序、处理器会对代码乱序执行等带来的问题。可以保证并发编程场景中的原子性、可见性和有序性。

