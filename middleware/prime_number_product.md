# for循环实践：生成10000以内的质数

分析：质数是只能被1和本身整除的正整数，最小质数是2，所以要对质数求余，除本身外，除以其他数都会有余数；

```java
int zs = 0;
x1:
for (int i = 2; i <= 100; i++) {
    x2:
    for (int j = 2; j <= i; j++) {
        if (i % j == 0 && i != 2 && i != j) {
            continue x1;
        } else {
            zs = i;
        }

    }

    System.out.print(zs+"\t");
}
```

这里为了方便调试，先以100以内的质数，用i当做被除数，j当做除数；两层循环都来搞定；j不断++，当i能被j整除就跳出外层循环，继续往下循环；但i等于2，或者i和j相等时 ；他们求余肯定也是0，要排除在外，因为质数也满足这个条件；让满足条件的数走另一个分支；把i存到zs变量这个分支，最后打印出来；  这里分别实践了for循环和循环标签；

继续提升难度，每4个换行一次，代码如何实现呢？我们增加一个计数器，当zs有4个时，就换行：

```java
int zs = 0;
int count = 0;
    x1:
    for (int i = 2; i <= 100; i++) {
        x2:
        for (int j = 2; j <= i; j++) {
            if (i % j == 0 && i != 2 && i != j) {
                continue x1;
            } else {
                zs = i;
            }

        }
        if (i == zs) {
            count++;
        }
        if (count % 4 == 0) {
            System.out.print(zs+"\n");
        }
        System.out.print(zs+"\t");
    }
```

一定要注意，这里打印是print，不是println，另外打印内容中zs在前，换行和tab符在后；

继续延伸，把这些质数存储到一个数组中怎么实现呢？

分析：我们事先不知道数组长度，可以存储到一个能自动增长的集合中，然后再转存到数组中，就行了；

```
private static int[] zhishu(int k) {
    int zs = 0;
    int count = 0;
   // int[] myarr =  new int[0];
    List<Integer> list = new ArrayList<>();
    x1:
    for (int i = 2; i <= 100; i++) {
        x2:
        for (int j = 2; j <= i; j++) {
            if (i % j == 0 && i != 2 && i != j) {
                continue x1;
            } else {
                zs = i;
            }

        }
        list.add(zs);
    }
    int [] arry = new int[list.size()];

    for (int i = 0; i <list.size() ; i++) {
         arry[i]=list.get(i).intValue();
    }
     return  arry;
 }
```

这样又能运用到集合的知识；

这个例子能让我们熟悉for循环的同时并学习到集合、计数器、打印等知识。