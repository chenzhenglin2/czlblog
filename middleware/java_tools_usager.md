# 常用工具和方法总结

## java中好用的方法

**判断字符串是否都是数字**

```java
if (null != str && 0 != str.trim().length() && str.matches("\\d*")) {     return true; } 
```



**计算字符串中 字母a出现的次数**

```java
int aLength = str.length()-str.replace("a","").length(); 
```



**打印json中的 key value  对（阿里的fastjson或者jackson 都行）**

```java
        JSONObject jsonObj = JSON.parseObject(jsonStr);
        for (Map.Entry<String, Object> entry : jsonObj.entrySet()) {
        System.out.println(entry.getKey() + ":" + entry.getValue());
        }  
```



**毫秒级的时间戳（timestamp 为long/Long类型的数据，如果不是提前转换）转换成Instant （单词意思为瞬间）类型**

```java
Instant.ofEpochSecond(timestamp) 输出：2017-03-29T02:56:02Z 

**把Instant 转成 LocalDateTime类型(instant 为Instant的实例）**

LocalDateTime.ofInstant(instant, TimeZone.getDefault().toZoneId()); 
```



**字符串的拼接**

```java
StringBuffer tmp = new StringBuffer();
for (String mystr1:strs22){
tmp.append(","+mystr1);
}
System.out.println(tmp.toString().substring(1)); 
```

思想就是先拼接，最后去掉第一个多余逗号，如果逗号放在后面，去掉末尾的逗号也可以

同时append后面还可以接append 如：tmp.append(",").append(str);



**字符串去重：**

```java
StringBuffer tmp2 = new StringBuffer();
for(int i=0;i<str.length();i++){
char letter = str.charAt(i);
int fistIndex = str.indexOf(letter);
if(fistIndex == str.lastIndexOf(letter) || fistIndex==i){
tmp2.append(letter);
	}
} 
```



**Aes编码与解码(不建议使用des算法了）**

```java
public static String enAes(String data, String key) throws Exception {
Cipher cipher = Cipher.getInstance("AES");
SecretKeySpec secretKeySpec = new SecretKeySpec(key.getBytes("UTF-8"),"AES");
cipher.init(Cipher.ENCRYPT_MODE,secretKeySpec); //ENCRYPT_MODE 为加密模式
byte[] enSecrets = cipher.doFinal(data.getBytes());
return Base64.getEncoder().encodeToString(enSecrets); // 来自import java.util.Base64;
}
public static String deAes(String data, String key) throws Exception {
Cipher cipher = Cipher.getInstance("AES");
SecretKeySpec secretKeySpec = new SecretKeySpec(key.getBytes("UTF-8"), "AES");
cipher.init(Cipher.DECRYPT_MODE, secretKeySpec);//DECRYPT_MODE 为解码模式
byte[] cipherTextBytes = Base64.getDecoder().decode(data);// 来自import java.util.Base64;
byte[] decValue = cipher.doFinal(cipherTextBytes);
return new String(decValue);
} 
```

md5（or 加盐） 加密和解密 很多工具中带有，工具类中有说明；其实md5加密，不管加几层盐，都很容易破解；开发过程用了Spring Security框架，直接用BCryptPasswordEncoder  方式加密和解密（matches），这个很好用和springboot 也无缝对接，最重要的是生成的密码基本破解不了。



**复杂型三元运算**

```java
 a=b==c?d:e 

如果b c相等，a赋值为d，否则赋值为e，现实应用：

error_count = null == error_count ? 1 : error_count + 1; 
```

java 赋值和布尔运算 都是先算后面的对象



**uuid的使用**

//解决文件名字问题：我们使用uuid; 

```
String filename = "ks-"+UUID.randomUUID().toString().replaceAll("-", ""); 
```



**Random  自动生成 一个限定上限范围随机数** 

原生的random方法，一般生成都是从0到某个上限（不含上限）数字，如：random.nextInt(5)  ；

生成一个指定上下区间的随机数 

```java
public static int rand(int min, int max) {
return random.nextInt(max) % (max - min + 1) + min;
} 
```



**把毫秒级的时间戳转换为LocalDateTime  以及 LocalDateTime LocalDate 类型和格式化**

如果想要时间戳转换成 LocalDateTime 必须先转换成Instant，然后通过Instant 转换成LocalDateTime，格式化就直接用DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss") 括号里的格式 可以自定义

```java
1，Instant  in  = Instant.ofEpochSecond(秒单位的时间戳，微秒/1000也行;这样就把时间戳转换成Instant了   Instant  in  = Instant.ofEpochMilli(微秒秒单位的时间戳） 
2，LocalDateTime ldt = LocalDateTime.ofInstant(Instant instant, ZoneId zone)  ;
ZoneId  可以通过TimeZone.getDefault().toZoneId来获取(java虚拟机的时区），也可以通过自行定义    		       TimeZone后，再toZoneld      如：TimeZone.getTimeZone("Asia/Shanghai");   
格式化： 写法：ldt.format(格式化pattern)： 实现：ldt.format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")) 
```





### java常见的数据结构增删改查方法

**1，数组**

遍历直接用for循环，如果多维数组，用多个for循环， 或者转换list 用foreach 或者stream流

增加、删除、替换操作：由于数组长度是固定的，增删就只能扩展为新的数组，  替换通过遍历查找 或者指定位置来实现



**2，链表**

如ArrayList  LinkedList

 增加 add  查找 get 删除是remove（参数为索引，或者对象）  替换为set（int dex,Object o）

Set 集合与链表类似，就是无 指定位置替换的方法set(index,object)

**3,栈**

查找 search 

  压入 push 

 弹出并删除顶部元素pop，  

查看顶部元素（不删除）peek



**4，队列**

增加add  和 offer 都行 

删除 remove  删除队列头  poll 也是删除队列头 如果队列为空  返回null

peek  查看队列头 不删除



**5， map**

删除所有  clear

遍历  foreach

查找某个 get

增加是put

删除是remove （）

 树、 堆 、 图   没有默认的统一接口 ，只能自己写接口 然后实现接口

**关于他们的长度**

数组的长度是length， 这个是属性，字符串length() 是方法，其他队 栈  链表等各种类型集合  的都是 size()





**数据转换注意事项**

```
(int)(System.currentTimeMillis()/1000)

 (int)System.currentTimeMillis()/1000 
```

这两个方法 结果差别很大，第一个得到微秒的时间戳后除以1000的结果 是long类型，强转为int

第二个是先转为int  丢失了很大的精度，又除以1000 放大了1000被，所有结果 就不正确了

以后所有计算结果进行转换 ，把计算过程用括号包起来。



**关于金额**

mysql 中金额用decimal（列默认值为0.00） ，对应java实体类属性是BigDecimal 类型，java中BigDecimal

都可以通过构造参数 携带int 、double、 long、String 其中的一种，就可以把他们转换成BigDecimal类型的。同理 BigDecimal 可以轻松转换成 int  double  long String (方法是.intValue()这种形式)  

但建议 只用String类型进行初始化

BigDecimal num12 = new BigDecimal("0.005");  

有加减乘除、绝对值 方法，方法名称分别是

add/subtract/multiply/divide abs

其中要注意的时 divide 时，要指定保留小数点位数，还可以指定数据保留原则，比如四舍五入参数是：ROUND_HALF_UP

大多时候 会编写一个不能实例化的类（变量是常量，方法是类方法），来处理金额计算

**BigDecimalUtils ，可以参考：**

```java 
public   static   double  add( double  v1, double  v2)       {
    BigDecimal b1 =  new  BigDecimal(Double.toString(v1));
    BigDecimal b2 =  new  BigDecimal(Double.toString(v2));
    return  b1.add(b2).doubleValue(); 
}
```





**A对象的属性copy到B对象上**

```java
public int update(Long id, A a) { 

​    B b= new b();

​    b.setId(id); 

​    BeanUtils.copyProperties(a, b); 
```

**注意事项：**b包含a对象所有属性,BeanUtils 为springboot框架提供的





**流式处理Stream**

博客有专门一章讲解，这里提一下数组、集合、map遍历、排序、各种操作 都可以用它来处理，效率和速度 没得说。



## 常用的工具（前面介绍的方法可以忘了，下面介绍的工具类基本都提供了）

### 分页工具pagehelper 

具体用法，不做介绍；  需要注意，这个相当于把传输对象包了一层，如果前端使用对象集合的话，需要用xx.list才可以。GitHub上有详细用法，引入这个工具后，源码中的注释也是中文的，都可以看到详细的用法。

### Apache提供的工具类

commons-lang3 

  **StringUtil**

非空白判断：

```
StringUtils.isBlank(" ")       = **true**;
```

可以直接使用以下方法获取拼接之后字符串：

```
StringUtils.join(["a", "b", "c"], ",")    = "a,b,c"
```

**DateUtils/DateFormatUtils**

```
// Date 转化为字符串*
DateFormatUtils.format(**new** Date(),"yyyy-MM-dd HH:mm:ss");
*// 字符串 转 Date*
DateUtils.parseDate("2020-05-07 22:00:00","yyyy-MM-dd HH:mm:ss");
```

除了格式转化之外，`DateUtils` 还提供时间计算的相关功能。

```
Date now = new Date();
// Date 加 1 天
Date addDays = DateUtils.addDays(now, 1);
// Date 加 33 分钟
Date addMinutes = DateUtils.addMinutes(now, 33);
// Date 减去 233 秒
Date addSeconds = DateUtils.addSeconds(now, -233);
// 判断是否 Wie 同一天
boolean sameDay = DateUtils.isSameDay(addDays, addMinutes);
// 过滤时分秒,若 now 为 2020-05-07 22:13:00 调用 truncate 方法以后
// 返回时间为 2020-05-07 00:00:00
Date truncate = DateUtils.truncate(now, Calendar.DATE);
```

#### commons-collections4

list为空判断

```
   if(CollectionUtils.isEmpty(list)){

}  
```

### commons-io 

#### FileUtils-文件操作工具类

```
// 拷贝文件*
File fileA = **new** File("E:\\test\\test.txt");
File fileB = **new** File("E:\\test1\\test.txt");
FileUtils.copyFile(fileA,fileB);
```

**还有GitHub/gitee上一个比较热门且强大并好用的  也是国内开发的java工具包  Hutool**  

<https://gitee.com/loolly/hutool>  gitee源码地址： 非常好用，里面有详细说明，上面罗列的工具类这里都可以找到。

墙裂推荐，基本上有它，其他工具类 就不用引入了，最重要的是 文档都是中文的；