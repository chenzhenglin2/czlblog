# 利用单向链表 进行递归的实践

## 原理阐述

  首先阐述一下什么叫递归呢，就是方法不断的自身调用，直至不满足条件，跳出这个递归循环；举个栗子：从前有座山，山里有座庙，庙里有个小和尚、小和尚让老和尚给他讲故事，然后老和尚开始讲故事：“从前有座山，山里有座庙，庙里有个小和尚、小和尚让老和尚给他讲故事，然后老和尚开始讲故事……”  这就是个递归，而且这是个死循环递归。 
  那正常的递归是什么样的呢，还是这个故事，稍作改动一下就可以了：从前有座山，山里有座庙，庙里有个5岁的小和尚、小和尚让老和尚给他讲故事，然后老和尚开始讲故事：“从前有座山，山里有座庙，庙里有个4岁的小和尚、小和尚让老和尚给他讲故事，然后老和尚开始讲故事……”  这个直至小和尚到1岁（因为0岁的小和尚还未出生），结束递归。

## 牛刀小试

```java
public class Hannio {

   private void hanTest(int n,char from,char mid,char to) {
      if(n<1) {
         System.out.println("输入有误");
      }else if(n==1) {
         System.out.println("把第1个箱子从"+from+"移到"+to);
      }else {
         //把n-1个箱子 移动到mid上,to作为过渡
         hanTest(n-1, from, to, mid);
         //把第n个箱子移动到to上
         System.out.println("把第"+n+"个箱子从"+from+"移动到"+to);
         //把n-1个箱子从mid 移动到to上，from作为过渡；
         hanTest(n-1, mid, from, to);
      }
   }
```

- 上面代码是简单实现一个汉诺塔程序，汉诺塔可以看作是一个比较典型的递归现象； 

首先总结规律，不管多少个箱子移动，只有两种情况：第一种 只有一个箱子，从第一根柱子直接移动到第三根柱子上。 第二种 有两个箱子，先把第一个箱子移动到中间的柱子，第二个箱子移动到第三根柱子上，最后第一个箱子移动到第三根柱子上，完成任务。 那如果有很多箱子呢，比如有n个箱子呢，这个可以把n-1个箱子看成第一个箱子，n是第二个箱子；这样就很好移动了；先用递归方法让n-1个箱子移动到第二根柱子上，然后第n个箱子移动到第三根柱子上。最后再用递归实现剩下的n-1个箱子从第二根柱子上移动到第三根柱子上。

通过上面例子，基本理解了递归的本质了，接下来用递归实现单向链表，实现增、删、查、显示功能。

## 单向链表的实现

```java
package com.zhengling.work;

//可以吧ListDemo 理解为链表类，初始化只有一个head 节点，其值为null；
public class ListDemo {
    //定义一个内部类，包含存放值的value 和存放下一个元素内存地址的next，可以理解为实现节点特性的类
    class Entry{
        int value;
        Entry next;

        //添加一个可以存放值的构造器
        public Entry(int value) {
            this.value = value;
        }
        //添加节点，如果当前节点的next为空，把节点增加到后面；如果不为空 使用递归，直至找到next为空的位置，加到此处。
        public  void add(Entry entry){
            if(this.next == null){
                this.next = entry;
            }else {
                this.next.add(entry);
            }
        }
        //打印节点中的值，如果当前节点的next不为空，递归打印 直至节点的next为空。
        public void show(){
            System.out.print(value+"------>");
            if(this.next != null){
                this.next.show();
            }
        }

        //查找值 如果当前节点的值和查找的值 相等 返回true
        //如果当前节点没有查找到，当前节点next不为空 采用递归一直往下查找
        //如果递归查询完了 还是没找到 则返回false；
        public boolean search(int i){
            if(this.value == i){
                return true;
            }else{
                if(this.next != null){
                    return this.next.search(i);
                }else {
                    return  false;
                }
            }
        }

        //如果当前节点的下一个节点 里面有要删除的值  进行判断
        //如果下一个节点的下一个节点 不为空 ，下一个节点=下下一个节点；
        //下下一个节点为空的话，自己让下一个节点=null；
        public  void delete(int i){
            if(this.next.value ==i){
                if(this.next.next != null){
                    this.next=this.next.next;
                }else {
                    this.next=null;
                }
            }else{
                this.next.delete(i);
            }
        }

    }

    //初始化一个起始节点。
    Entry head;
    //如果起始节点为空，则把这个节点放到起始节点中。如果起始节点不为空 在head后面调用添加节点的方法；
    public void addValue(int i){
        Entry entry = new Entry(i);
        if(this.head == null){
            this.head = entry;
        }else {
            this.head.add(entry);
        }
    }

    //如果起始节点不为空 就打印节点，起始节点为空了 代表链表是空的 没有打印的必要。
    public void showValue(){
        if(this.head != null){
            this.head.show();
        }
    }

    //查找：如果起始节点中的value 就等于查找的值，返回true；
    //否则 就判断起始节点是否还有下一个节点，如果有，就让head调用查找查找此值的方法；
    //如果没有查找到此值 就返回false；
    public  boolean searchValue(int i){
        if(this.head.value == i){
            return  true;
        }else{
            if(this.head.next != null){
                return  this.head.search(i);
            }else {
                return  false;
            }
        }
    }


    //首先判断 这个链表有没有这个类，如果有再进行删除操作
    //如果起始节点中有这个值，然后起始节点有下一个节点  让起始节点的值等于其next，否则起始节点=null
    //如果起始节点没有这个值，让起始节点调用查找值的方法。
    public void deleteValue(int i){
        if(this.searchValue(i)){
            if(this.head.value == i){
                if(this.head.next != null){
                    this.head =this.head.next;
                }else{
                    this.head = null;
                }
            }else{
                this.head.delete(i);
            }
        }
    }

}
//测试类
class ListDemoTest{
    public static void main(String[] args) {
        ListDemo  ld  = new ListDemo();
        ld.addValue(1);
        ld.addValue(3);
        ld.showValue();
        System.out.println();
        ld.addValue(4);
        ld.showValue();
        System.out.println();
        System.out.println(ld.searchValue(3));
        System.out.println(ld.searchValue(5));
        ld.deleteValue(3);
        ld.showValue();
    }
}
```

具体实现过程和思路已在代码中 注释。