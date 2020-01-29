# for和while相互转换

- 先看段代码：

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

```java
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

但while循环语句更容易控制，只需验证括号里面布尔表达式是否成立就行，至于判断条件的变化，可以放到循环体里面来实现，并能灵活的加上限制条件（if),如实现 打印如下形状的数据，用while循环更合适；

```
1	
2	3	
4	5	6	
7	8	9	10	
11	12	13	14	15	
```

- 这种漏斗形式的数据，先总结规律，再来代码实现；上面形式数据特点是：每一行的元素个数和行号一样多；  那就说明当前行和当前列是一样多的，换句话说：当前行 最后一列是换号符的话，这个列号和下一行 行号是一样的；

- 思路有了，代码就好实现；需要三个变量来实现上面功能，一个供输出的变量index，一个行变量line，一个列变量coumn；当前行最后一列 和下一行行号相等的话，此列输出换号符；并让列重1开始，行号+1；继续打印i，然后列号+1；循环判断行号列的关系，一旦相等  就在此列位置输出换行符； 但要排除一种情况，就是行和列 都是1时，如果此时也输出换号的话，相等于第一行 第一列是个换行符 ；而1打印到第二行了。
- 代码实现：

```java
int line = 1;
int coumn = 1;
int index = 1;
while (index<=15){
    if (line == coumn) {
        if (index != 1) {
            System.out.println();
        }
        coumn=1;
        line++;
    }
    System.out.print(index+"\t");
    index++;
    coumn++;
}
```

while 布尔表达式的灵活性还有用变量来表示，如 boolean  flag = true； while(flag){循环体}，在循环体中，出现某种情况让flag值变为false，即可实现中断循环的功能；