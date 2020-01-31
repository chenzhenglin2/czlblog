# 枚举与switch结合实践

- 枚举，可以罗列出有限序列集合，比如最常见的周一至周日，switch分支判断语句，这里可以做一下结合，这样能掌握两个知识点。

  ```java
  package com.zhengling.work;
  //  枚举中也可以使用构造器，默认是使用private修饰的； 枚举中每个元素 都是按照构造器（格式）生成的对象；
  public enum WeekEnum {
      MON("1001","番茄鸡蛋",12.0),
      TUE("1002","鱼香肉丝",24.0),
      WED("1003","地三鲜",18.0),
      THU("1004","红烧肉",35.0),
      FRI("1005","宫保鸡丁",30.0),
      SAT("1006","麻辣香锅",32.0),
      SUN("1007","家常豆腐",15.0);
      private  String no;
      private  String name;
      private  double price;
  
      WeekEnum(String no, String name, double price) {
          this.no = no;
          this.name = name;
          this.price = price;
      }
  
      public void setNo(String no) {
          this.no = no;
      }
  
      public void setName(String name) {
          this.name = name;
      }
  
      public void setPrice(double price) {
          this.price = price;
      }
  
      public String getNo() {
          return no;
      }
  
      public String getName() {
          return name;
      }
  
      public double getPrice() {
          return price;
      }
  }
  ```

  上面代码创建了一个常见的周一至周日的枚举，在每天推出不同的特价菜；然后定义一个构造方法（默认是用private修饰的构造方法，只能用private修饰）、getter、sett等；枚举中的每个元素都要按照这个构造方法来实现。

  然后我们在用switch 来一一判断：

  ```java
  package com.zhengling.work;
  
  //import static org.junit.jupiter.api.Assertions.*;
  //枚举和switch case的学习
  class WeekEnumTest {
      public static void main(String[] args) {
          discountMenu(WeekEnum.SAT);
  
      }
  
      public static void discountMenu(WeekEnum we) {
          switch (we){
              case MON:
                  System.out.println("今日特价菜是："+we.getName());
                  break;
              case TUE:
                  System.out.println("今日特价菜是："+we.getName());
                  break;
              case WED:
                  System.out.println("今日特价菜是："+we.getName());
                  break;
              case THU:
                  System.out.println("今日特价菜是："+we.getName());
                  break;
              case FRI:
                  System.out.println("今日特价菜是："+we.getName());
                  break;
              case SAT:
                  System.out.println("今日特价菜是："+we.getName());
                  break;
              case SUN:
                  System.out.println("今日特价菜是："+we.getName());
                  break;
              default:
                  throw new IllegalStateException("Unexpected value: " + we);
          }
  
      }
  }
  ```

  从这个案例，可以迅速掌握enmu和switch语法。随着jdk最新版本推出，会逐渐丰富switch写法的；最新的jdk13已经支持 switch里面支持匹配多条件、携带返回值、最新的lamda格式的写法，以后会越来越好用。

