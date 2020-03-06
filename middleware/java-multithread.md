# 如何利用多线程交替输出奇偶数

## 传统模式

用两个线程分别输出奇数、偶数

- 思路：两个线程，一个输出奇数，一个输出偶数；说明这两个线程公用一个数据池。一个线程输出数据时，另一个线程不能输出，需要等另一个线程完成任务后，再执行输出任务。所以我们要抽象出一个数据池类，我们线程都调用数据池类型的同一个对象，然后用synchronize 同步锁配合wait、notifyAll等方法限制只能一个进程消耗数据池。

- 代码：

  ```java
  package com.zhengling.work;
  
  public class AlternatePrintNum {
      public static void main(String[] args) {
          NumberPool np = new NumberPool(1);
          Thread t1 = new Thread(new OddProcessor(np));
          Thread t2 = new Thread(new OddProcessor(np));
          t1.setName("进程1");
          t2.setName("进程2");
          t1.start();
          try{
              Thread.sleep(1000);
          }catch (Exception e){
              e.printStackTrace();
          }
          t2.start();
      }
  
  }
  class  NumberPool{
      int index;
  
      public NumberPool(int index) {
          this.index = index;
      }
      public synchronized void printNum(){
          System.out.println(Thread.currentThread().getName()+"----->"+(index++));
          try {
              Thread.sleep(500);
              this.notifyAll();
              this.wait();
          } catch (InterruptedException e) {
              e.printStackTrace();
          }
      }
  }
  
  class OddProcessor implements Runnable{
      NumberPool np;
  
      public OddProcessor(NumberPool np) {
          this.np = np;
      }
  
      @Override
      public void run() {
          while (true){
              np.printNum();
          }
      }
  }
  ```



- 注解：OddProcessor实现了Runnable接口，其类型对象可以添加到进程中，构造器中有数字池参数。

  run方法里面直接调用数字池对象的printNum（打印数字）方法，printNum方法用synchronize修饰，同时只能被同一个对象的一个线程调用。调用完了，唤醒其他线程、自己则无限等待，直至被唤醒。

 ## lamda表达式 实现接口

- 这就是典型的生产者与消费者模式，既然jdk8里面有了lamda表达式，那么OddProcessor 这个线程在调用时再用lamda表达式实现就好；改进后线程操作的操作方法。

 ```java
  NumberPool np = new NumberPool(1);
      new Thread(()->{while (true)np.printNum();},"进程1").start();
      try {
          Thread.sleep(1000);
      } catch (InterruptedException e) {
          e.printStackTrace();
      }
      new Thread(()->{while (true)np.printNum();},"进程2").start();
  
  }
 ```

##   传统的synchronize和juc里面的Lock 来分别实现交替打印字母与数字

难度再次升级 、使用传统的synchronize和juc里面的Lock 来实现交替打印字母与数字

```java
package com.zhengling.work;

import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

public class AlternatePrintNumAndLetter {

    static boolean flag = true;
    public static void main(String[] args) {

        System.out.println("第一种方法 传统的synchronized");
        final Object obj =  new Object();
        new Thread(()->{
            while (flag){
                synchronized (obj){
                    for(char letter='a';letter<='z';letter++){
                        System.out.println(Thread.currentThread().getName()+"----->"+letter);
                        obj.notifyAll();
                        try {
                            obj.wait();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                }
            }

        },"t1").start();

        new Thread(()->{
            while(flag){
                synchronized (obj){
                    for(int i = 1;i<=26;i++){
                        System.out.println(Thread.currentThread().getName()+"----->"+i);
                        obj.notifyAll();
                        try {
                            obj.wait();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                }
            }

        },"t2").start();

        try {
            Thread.sleep(10);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("===========================================");
        System.out.println("===========================================");
        System.out.println("===========================================");

        System.out.println("第二种方法 用JUC中的Lock 和 Condition 接口");
        ReentrantLock lock = new ReentrantLock();
        Condition digitLock = lock.newCondition();

        new Thread(()->{
            while(flag) {
                for (int i = 1; i <= 26; i++) {
                    lock.lock();
                    System.out.println(Thread.currentThread().getName()+"----->"+i);
                    digitLock.signalAll();
                    try {
                        digitLock.await();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    lock.unlock();
                }
            }
        },"t3").start();

        new Thread(()->{
            while(flag) {
                for (char l = 'a'; l <= 'z'; l++) {
                    lock.lock();
                    System.out.println(Thread.currentThread().getName()+"----->"+l);
                    digitLock.signalAll();
                    try {
                        digitLock.await();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    lock.unlock();
                }
            }
        },"t4").start();
       try {
            Thread.sleep(10);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        flag=false;

    }
}
```



从代码可以看出juc中的Lock 、Condition  ，也能实现锁与唤醒功能。与传统的synchronize比较，就像手动挡的车与自动挡的车；juc虽然像手动车，但效率和安全以及逻辑明显提升。 

代码中synchronize中wait()和notifyAll() 与之对应的是 await() 和 signalAll() ，但后面这两个必须包含在lock与unlock代码中间。

## 唤醒指定线程实现交替打印 数字、小写字母、大写字母

继续拓展，利用signal() 方法 指定唤醒某个线程，如实现交替打印 数字、小写字母、大小字母

```java
package com.zhengling.work;

import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class MyLockDemo {
    public static void main(String[] args) {
        Lock  lock  = new ReentrantLock();
        Condition condition1 = lock.newCondition();
        Condition condition2 = lock.newCondition();
        Condition condition3 = lock.newCondition();
        new Thread(()->{
            for (int i = 0; i <26 ; i++) {
                lock.lock();
                System.out.println(Thread.currentThread().getName()+"--->"+i);
                condition2.signal();
                try {
                    condition1.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    lock.unlock();
                }
            }
        },"t1").start();

        new Thread(()->{
            for (char i = 'a'; i <='z' ; i++) {
                lock.lock();
                System.out.println(Thread.currentThread().getName()+"--->"+i);
                condition3.signal();
                try {
                    condition2.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    lock.unlock();
                }
            }
        },"t2").start();


        new Thread(()->{
            for (char i = 'A'; i <='Z' ; i++) {
                lock.lock();
                System.out.println(Thread.currentThread().getName()+"--->"+i);
                condition1.signal();
                try {
                    condition3.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    lock.unlock();
                }
            }
        },"t3").start();

    }
}
```

需要注意的是 ：signal() 唤醒特定线程 一定要在要执行的操作后面； 一般固定格式： 判断 ->加锁->执行->唤醒某个线程->置为等待状态 ->释放锁