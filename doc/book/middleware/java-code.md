# 关于数组位置调整和元素出现次数统计代码

## 数组位置调整

**需求：调整数组，正数放在左边 负数放在右边，0在中间相对位置不变**

```java
public static int[] tzsx(int[] myarr) throws ArrayIndexOutOfBoundsException {
	   int[] newarr = new int[myarr.length];
	   int zhengshu = 0;
	   int zerocount = 0;
	   int count = 0;
	   for(int a:myarr){
	     if(a>0){
	      zhengshu++; //计算出正数的总个数
	     }
	     if(a == 0){
	      zerocount++;//计算出0的总个数
	     }
	   }  

	  int nofushu=zhengshu+zerocount;
	  for(int i=0;i < myarr.length;i++){
	    if(myarr[i] > 0){
	      newarr[count] = myarr[i];  //把正数放到新数组中，索引从0开始
	      count++;
	    }
	    else if(myarr[i] == 0){
	      newarr[zhengshu] = myarr[i];//把0放到新数组中，索引从正数总个数开始
	      zhengshu++;
	    }
	    else{
	      newarr[nofushu]=myarr[i];//把负数放到新数组中，索引从(正数+0)总个数开始	
	      nofushu++;	      
	    }

	  }
	  return newarr;
	}
```
这是之前写的代码（开发总会觉得之前代码写的像屎），发现繁琐且低效，用java8中的Stream流处理，就几行代码就能搞定。

```java
public static int[] sztz(int[] myarr) throws ArrayIndexOutOfBoundsException {
    int[] positive = IntStream.of(myarr).filter(x->x>0).toArray();
    int[] zero = IntStream.of(myarr).filter(x->x==0).toArray();
    int[] negative = IntStream.of(myarr).filter(x->x<0).toArray();
    int[] arrays = new int[positive.length+zero.length+negative.length];
    System.arraycopy(positive,0,arrays,0,positive.length);
    System.arraycopy(zero,0,arrays,positive.length,zero.length);
    System.arraycopy(negative,0,arrays,positive.length+zero.length,negative.length);
    return arrays;
}
```

改用IntStream操作，按规则形成的数组，再逐一合并到一个新数组；经验证效率提升明显，效果也一样。唯一需要注意的，这个流最好不要多线程并行，避免元素位置顺序打乱。

## 打印数组中多次出现的元素

**需求：打印出数组中元素出现次数大于2的元素和次数**

```java
public static void countStr(String[] arr) {
	Map<String, Integer> myMap=new HashMap<>();
	for(String str:arr) {
		Integer num=myMap.get(str);
		myMap.put(str, num==null?1:num+1);
	}
	Iterator<String>  it=myMap.keySet().iterator();
	while (it.hasNext()) {
		Object key =it.next();
		if(myMap.get(key)>1) {
			System.out.println(key+"\t total view "+myMap.get(key)+" times ");
		}
		
	}
} 
```
实现方式：把数组元素插入到hashmap中，元素作为key，出现次数作为value，最后遍历出来，判断一下，大于2次，就打印。或者在遍历时直接把map转换成Set<Map.Entry<K,V>>,  然后逐一取出Map.Entry  中K和V 就行了。