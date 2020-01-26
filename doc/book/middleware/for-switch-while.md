# for和while相互转换

先看段代码：

```java
for (int i = 1; i <10 ; i++) {
    for (int j = 1; j <= i ; j++) {
        System.out.print(j+"\t");
    }
    System.out.println();
}
System.out.println("-------->");
//转换成while循环
int k = 1;
while (k < 10){
    int n =1;
    while (n <= k){
        System.out.print(n+"\t");
        n++;
    }
    System.out.println();
    k++;
}
```

上面代码效果一样，都是输出（乘法口诀外层）：

```
1	
1	2	
1	2	3	
1	2	3	4	
1	2	3	4	5	
1	2	3	4	5	6	
1	2	3	4	5	6	7	
1	2	3	4	5	6	7	8	
1	2	3	4	5	6	7	8	9
```

for循环的格式是：for([初始化];[判断条件];[条件改变方式]){循环体}  其中()里面除了两个引号，其他不是非必须的 while更简单就括号里面判断条件；互相转换规律时，把for循环里面初始化写到外面，while括号里面只写判断语句，条件改变方式写到while循环体中，这样就可以达到for循环的效果；