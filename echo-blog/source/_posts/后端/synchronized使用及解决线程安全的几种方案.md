---
title: Synchronize
categories:
- 后端
tags: 线程安全
date:
---


> 说明：单线程是不存在线程安全问题的；但是多线程则可能会存在线程安全问题,所以解决线程安全问题是针对多线程而已的；

解决方案如下：
1. 同步代码块：在代码块上声明，也就是使用于方法内部
    ~~~
   synchronized(锁对象) {
   
   }

   特别注意:针对同步代码块而已，锁对象可以是任意对象，但是对于多线程而 
           已，该锁对象必须保持一致，否则会出现线程安全问题；
   用法demo:
   public class SynchronizedTest() {
      synchronized(Synchronized.class){
          //业务逻辑
      }
   }
2. 同步方法：使用synchronized关键字修饰的方法
   ~~~
   a、非静态同步方法：
   public synchronized SynchronizedMethodTest() {
	//业务逻辑
   }
   注意：非静态同步方的锁对象是this，代表实例对象本身，
         说白了也就是类名.class,但是this并不会在代码中直接展示出来，只 
         是作为一个隐藏的锁对象       
   b、静态同步方法：-----通过static关键字修改
   public static synchronized() {
      //业务逻辑
   }
   注意：静态同步方法的锁对象是 类名.class;
3. ※同步锁: 使用的是Lock接口，需要手动获取锁和释放锁操作
   ~~~
   //创建锁对象
    private Lock lock = new ReentrantLock();
    //测试方法
    public void testLock(Thread thread) throws InterruptedException{
     if (lock.tryLock(3000, TimeUnit.MILLISECONDS)) { //尝试获取锁，修改等待时间
          try {
               System.out.println("当前线程"+ thread.getName() + "获取到了锁");//打印当前锁名称并设置睡眠时间
               Thread.sleep(4000); //设置睡眠时间，超过修改时间，这样后面的线程就无法获取到锁对象了

                } catch (InterruptedException e) {
                    System.out.println("当前线程"+thread.getName() + "发生了异常释放锁");  //若捕捉到异常，则打印线程名称+执行碰到异常
                } finally {
                    System.out.println("当前线程"+ thread.getName() + "执行完毕"); //finally打印线程名称执行完毕
                    lock.unlock(); //释放锁对象

                }
            }else {

                System.out.println("当前线程" + thread.getName() + "未成功，锁占用");
            }
    }
   
参考文档：https://blog.csdn.net/gm371200587/article/details/88173030
