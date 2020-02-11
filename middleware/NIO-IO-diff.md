# NIO和IO遍历指定目录效果对比

NIO是jdk7引入的，IO是1.6版本之前就有的；现在就分别利用NIO和IO分别遍历一下某个目录下所有文件，对比写法、效率；

利用IO的File遍历目录：

```java
public static void showPath(String dir){
    File files = new File(dir);
    if (files.isDirectory()) {
        for (File file: files.listFiles()
             ) {
            System.out.println(file.getAbsolutePath());
            showPath(file.getAbsolutePath().toString());
        }
    }else{
        return;
    }
}
```

通过递归来实现遍历所有文件

进一步 借助NIO中Path 遍历目录

```java
public static void showDir(String dir){
    File files = Paths.get(dir).toAbsolutePath().toFile();
    if(files.isFile()){
        return;
    }
    for (File file:files.listFiles()
         ) {
        System.out.println(file);
        showDir(file.toString());
    }
}
```

代码优化的不太明显，这里只是做一下过渡，Paths.get(dir) 返回的是Path类型变量；翻阅Path源码发现它只是个接口，没有具体类去实现它的接口。刚接触时比较迷惑，打印一下此对象的getClass,得到的是“class sun.nio.fs.WindowsPath”   查阅资料发现Path 具体类型是由对象决定的；对象是由工厂模式（后续会介绍）创造出来的；之所以要引入这个话题，是建议大家最好让对象持有接口，而不是具体实现类；如需要把数据插入到一个列表中，实例这个列表时，最好用：List<限定类型> list = new  ArrayList<>();甚至在使用时，再new出来；随着lamda的引入，接口没有实现的方法，用的时候再根据实际情况编写也很方便。 

再优化一步，采用NIO的Files.walk方法，借助Stream流 直接逐一把目录打印出来：

```java
public static void printDir(String dir){
    Path path = Paths.get(dir);
    try {
        Files.walk(path).forEach(System.out::println);
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

核心代码只有一句，更快更方便的实现遍历目录的目的。

汽车为你造好了，没必要再骑车上高速了。

接下来会在[利用NIO读写文件](middleware/use-NIO-read-write.md)，中介绍NIO这个汽车比IO自行车的优点，若已掌握IO知识更好，这样以后文件操作大都用NIO，偶尔IO辅助一下。