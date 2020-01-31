# 利用lamda表达式实现接口方法

若想直接使用 接口中方法，在jdk8之前的做法有：1，编写一个类实现这个接口的（抽象）方法；2，或者直接在创建对象时 采用匿名类。

- 接口：

```java
package com.zhengling.work;

public interface DecoratorF {
    void m1();
    
}
```

- 里面有个方法m1;如果想调用m1方法，必须先实现这个接口的m1方法。用匿名内部类来实现：

```java
 DecoratorF df = new DecoratorF() {
            @Override
            public void m1() {
                System.out.println("内部匿名类实现m1方法");
            }
        };
        df.m1();
```

- 这样就可以直接调用m1方法；或者搞一个内部类来实现这个接口：

```java
static class B  implements DecoratorF{
        @Override
        public void m1() {
            System.out.println("静态内部类实现接口的m1方法");
        }
    //定义为静态内部类 方便在main方法中调用
    }
```

然后用多态的特点 调用m1方法：

```java
DecoratorF def = new B();
def.m1();
```

上面不管哪种写法都显得比较繁琐，而且效率不高。jdk8推出了lamda表达式，采用这种形式能快速简化代码；说句题外话Python中lamda已经推出好久了，以后编程语言会大一同，很多优秀规范、表达式会互相借鉴。像jdk13中也引入了三个双引号 +内容+三个双引号，可以原样保持内容的规范、switch表达式借鉴swift语言写法。总之就是代码上越来越简单。

lamda表达式写法

```java
public static  void  doM1(DecoratorF df){
    df.m1();
}
```

之所以定义为static类型方法，是方便在main中调用和测试。 上面方法是直接调用了m1方法，在main方法中，调用时 用lamda表达式实现接口中的方法。

```java
doM1(()-> System.out.println("lamda表达式实现的抽象方法"));
```

doM1(这个里面其实是DecoratorF 类型参数，这个参数并且实现了m1方法)，由于m1是没有参数的方法，可以直接写成()->{} ，由于方法体就是一句话，{}可以省略。如果有参数可以写成 (parm1,parm2)->{}这种形式。

- 不适用地方，接口中有多个方法的。