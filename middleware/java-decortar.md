# 如何利用装饰者模式进行方法扩充

- 适用场景说明

    在编码过程中经常会遇见到原有方法需要扩充的情况，要么在原有类上进行扩充、要么写一个继承类进行扩充；  但这样做都有一定的弊端；原有类进行扩充会返工 导致其他问题，继承类进行扩充耦合度太高；这时候装饰者模式就派上用场了。

  如有一个类DecoratorA，里面有一个m1方法：

  ```java
  public class DecoratorA  {   
      public void m1() {
          System.out.println("我是A");
      }
  }
  ```

DecoratorB对其扩充：

```java
public class DecoratorB{
    DecoratorA da;
    public DecoratorB(DecoratorA da){
        this.da=da;
}


public void m1() {
    System.out.println("我是B,我对A类的方法进行扩展1");
    da.m1();
    System.out.println("我是B,我对A类的方法进行扩展2");
}


}
```

这样也能完成对DecoratorA类的m1方法扩充，new一个DecoratorB 类型的对象，构造参数是DecoratorA  类型就行；  用DecoratorB 类型对象调用m1，就实现调用扩充后的m1方法；但限制条件DecoratorB和DecoratorA的m1方法名称和写法、参数要一致；

为了规范，避免方法参数不一致导致的异常，可以再编写一个抽象类（接口也可以），里面只有一个抽象方法m1：

```java
package com.zhengling.work;

public abstract class DecoratorFather {
    public abstract  void  m1();
}
```

然后让DecoratorB和DecoratorA去继承这个抽象类即可；DecoratorB 构造器中参数类型可以写成DecoratorFather的，利用java多态特性，到时候new 一个DecoratorB ，构造参数可以填写子类型DecoratorA；

改进后的DecoratorA：

```java
package com.zhengling.work;

import org.junit.Test;

public class DecoratorA extends DecoratorFather {
    @Override
    @Test
    public void m1() {
        System.out.println("我是A");
    }
}
```

DecoratorB :

```java
package com.zhengling.work;
import org.junit.Test;
public class DecoratorB extends DecoratorFather {

    DecoratorFather df;
    public DecoratorB(DecoratorFather df){
        this.df=df;
}

    public void m1() {
        System.out.println("我是B,我对A类的方法进行扩展1");
        df.m1();
        System.out.println("我是B,我对A类的方法进行扩展2");
    }
}
```

再编写一个测试类：

```java
package com.zhengling.work;

public class DecortarTest {

    public static void main(String[] args) {
        DecoratorB db = new DecoratorB(new DecoratorA());
        db.m1();


    }
}
```

输出结果中可以看出DecoratorB 对DecoratorA的m1方法进行了扩展；这种装饰者模式在IO流控制类中多有实践，如某一个输出流中关闭了，所有的流都会同步被关闭。