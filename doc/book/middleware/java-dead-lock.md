# 利用synchronized关键字了解死锁

- synchronized是java关键字从字面上理解是同步的意思，它的作用是限制多线程的，使多线程暂时变成一个线程，确保线程的安全；synchronized代码块（包括方法、某个对象）只能一个线程调用；待这个线程调用完成后，其他线程才能调用此代码块； 就像生活中 ，第一个线程执行遇到synchronized，它把门锁上了，等它执行完了代码；再打开门，让其它线程可以进入代码块中。

- 我们可以利用synchronized这种机制，编写一个死锁；死锁的目的在于我们了解死锁的形成和如何解决。

- 来个最简单死锁；线程甲 锁住对象a， 然后准备再锁住b,完成后释放a;  线程乙先锁住对象b，再锁a，最后释放b；  这种场景就会进入一个僵局，死锁出现了。

```java
package com.zhengling.work;

public class DeadLock {
    public static void main(String[] args) {
        String o1 = "a";
        String o2 = "b";
        Processor1 p1 = new Processor1(o1,o2);
        Processor2 p2 = new Processor2(o1,o2);
        p1.start();
        //p1中会先锁住o1对象，2s后，接着去拿o2的对象锁（o1还在锁住状态）
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        p2.start();
        //o2对象锁已经被p2线程拿走（没有被归还）p2接着去拿o1对象锁，o1对象锁线程p1一直未归还；
        //所以就出现了死锁；p1对象只有拿到o2对象锁后和解除锁定o2后，才能继续往下走，归还o1的
        //对象锁；但o2对象锁已经被p2线程拿走，p2线程同样的 待拿到o1对象锁后 才能往下走，
        //归还o1、o2的对象锁
    }
}
class Processor1 extends Thread{
    String str1;
    String str2;

    public Processor1(String str1, String str2) {
        this.str1 = str1;
        this.str2 = str2;
    }
    public void run() {
        synchronized (str1){
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            synchronized (str2){
                System.out.println("P1's str2");
            }
        }
    }
}
class Processor2 extends Thread{
    String str1;
    String str2;

    public Processor2(String str1, String str2) {
        this.str1 = str1;
        this.str2 = str2;
    }
    public void run() {
        synchronized (str2){
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            synchronized (str1){
                System.out.println("P1's str2");
            }
        }
    }
}
```

上面代码 就是非常简单的死锁案例，平时写代码时一定要注意，如果锁定多个对象，确认是否和其他线程形成死锁；可以利用优先级来确定锁定的目标或拿到锁的顺序。  

