# 如何利用多线程交替输出奇偶数

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