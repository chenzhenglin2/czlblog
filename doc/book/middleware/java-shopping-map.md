# 运用简单的超市购物车系统，理解重写equals、hashcode的意义，以及map的学习

## 商品类：

- 先编写一个商品类，要有编号、名称、价格三个成员变量，然后把getter、setter都写好（可以用ide生成）

```java
package com.zhengling.work;

import java.util.HashMap;
import java.util.Map;

public class Shopping {
    String no;
    String name;
    double price;
    public Shopping() {
    }

    public Shopping(String no,String name,double price) {
        this.name = name;
        this.no = no;
        this.price = price;
    }

    public void setName(String name) {
        this.name = name;
    }


    public  void setNo(String no){
        this.no=no;
    }

    public void setPrice(double price) {
        this.price = price;
    }

    public String getName() {
        return name;
    }

    public String getNo(){
        return  no;
    }

    public double getPrice() {
        return price;
    }

    @Override
    public String toString() {
        return "Shopping{" +
                "编号：'" + no + '\'' +
                ", 名称是：'" + name + '\'' +
                ", 单价是：" + price +
                '}';
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;

        Shopping shopping = (Shopping) o;

        return no.equals(shopping.no);
    }

    @Override
    public int hashCode() {
        return no.hashCode();
    }
}
```

这个商品类，里面重写了toString方法，方便打印，构造器必须有编号、名称、价格三个参数。为什么要重写equals方法呢，所有类默认都是继承Object类的，Object的equals方法是比较两个对象的内存地址，如果相同则是同一个对象，我们这里商品编号相同 就是同一个东西；

Object里面判断方法：

`if (this == o) return true`

- 重写equals只需在上面代码基础上继续丰富就行。

  ```java
  if (o == null || getClass() != o.getClass()) return false;

        Shopping shopping = (Shopping) o;

        return no.equals(shopping.no);
        
  ```

还有一种常见的写法：

```java
if (o instanceof Shopping) {

    Shopping shopping = (Shopping) o;

    return no.equals(shopping.no);
}
```

推荐第一种写法，第一种写法限制对象必须是当前类型及Shopping类型。第二种方法风险在于如果有两个类都继承了Shopping，都有成员变量弄，且都没有重写equals，就会判断失误。

- 为什么要同时重写hashCode方法呢，因为这些类 new出来的对象，存放到HashSet、HashMap等集合、map中；首先比较的是hash值（散列值），如果没有重写hashCode，就很容易出现hash值相同的数据被覆盖掉的情况。hashCode 最简单的一个写法就是：返回用于比较两个对象是否相等的成员变量的hash值 

```java
public int hashCode() {
    return no.hashCode();
}
```

也可以参照String Integer源码 里面hashCode的写法。

## 购物车

购物车用HashMap表示，key表示存放商品，value表示数量。

```java
import java.util.HashMap;
import java.util.Iterator;
import java.util.Map;
import java.util.Set;

public class ShoppingCart {
    Map<Shopping,Integer> map = new HashMap<>();
    public void add(Shopping s){
        add(s,1);
    }
    public  void add(Shopping s,Integer num){
        map.put(s,num);
    }
    public  void deleteShopping(Shopping s){
        deleteShopping(s,1);
    }
    public  void deleteAllShopping(Shopping s){
        map.remove(s);
    }
    public  void deleteShopping(Shopping s,Integer num){
        if(num < map.get(s)){
            map.put(s,map.get(s)-num);
        }else{
            map.remove(s);
        }

    }
    public void clear(){
        map.clear();
    }
    public void printTotalItem(){
        Set<Shopping> set  = map.keySet();
        if(set.size() == 0) System.out.println("购物车为空！！！！！");
        Iterator<Shopping> it = set.iterator();
        Shopping sp;
        double total=0.0;
        while (it.hasNext()){
            sp=it.next();
            System.out.println(sp.no+"\t"+sp.name+" 单价："+sp.price+"\t数量："+map.get(sp)+"\t 小计："
                    +(double)(sp.price*map.get(sp).intValue()));
            total += (double)(sp.price*map.get(sp).intValue());
        }
        System.out.println("\t\t\t\t\t\t\t\t合计："+total+"元");
    }
    public void printShopping(){
        Set<Map.Entry<Shopping,Integer>> myset = map.entrySet();
        if(myset.size() == 0) System.out.println("购物车为空");
        Shopping p;
        int number;
        double total=0.0;
        for (Map.Entry<Shopping,Integer> ent: myset) {
           p= ent.getKey();
           number=ent.getValue().intValue();
            System.out.println(p.name+"的单价是："+p.price+"\t 数量是 "+number+"\t 小计："+(double)(p.price*number));
            total += (double)(p.price*number);
        }

        System.out.println("\t\t\t\t\t\t 商品总价是："+total);
    }

}
```

里面有增加商品、清除商品、清空购物车、打印清单等功能。



## 最后写一个简单的测试方法

```java
package com.zhengling.work;

public class ShoppingCartTest {
    public static void main(String[] args) {
        Shopping sp1 = new Shopping("1001","西瓜",1.0);
        Shopping sp2 = new Shopping("1002","黄瓜",12.0);
        Shopping sp3 = new Shopping("2001","苹果",2.0);
        Shopping sp4 = new Shopping("3001","菠萝",12.0);
        System.out.println(sp1);
        System.out.println("-----");
        ShoppingCart spc = new ShoppingCart();
        spc.add(sp1);
        spc.add(sp2,2);
        spc.add(sp3,4);
        spc.add(sp4);
        //spc.printShopping();
       spc.printTotalItem();
        System.out.println();
        spc.printShopping();

    }
}
```

然后运行，并验证通过。