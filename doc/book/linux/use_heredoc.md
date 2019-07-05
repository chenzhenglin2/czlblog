# heredocument 的巧用

我们经常使用

```
cat  >(或>>) xxx<<EOF

context ……

EOF
```

来把内容添加某个文件中，这就是heredocument 的一种用法，但heredocument 用法不仅仅限制于此



## 情景1：

 我们需要先登录mysql后 才能执行sql脚本 这个时候 就可以利用heredocument 

```
mysql -uroot -pxxxx <  xxx.sql
```



## 情景2：

:执行某个脚本实现登录后，然后执行相关操作

可以借鉴here document ，写成脚本   

```
#!/bin/bash

  ./xxx.sh  <<eof

要执行的命令

退出操作如quit或exit  

#这个根据具体登录软件，退出命令有关如果数据库 大部分可以写quit也有exit(），或其他写法

eof
```

- xxx.sh 也可以是可执行的二进制文件

## 注意：

另外eof包含的内容，含有变量的话 ，不被替换 可以写作

```
./xxxx.sh  <<'eof'

echo  $!

xxx

echo  $?

ttt

eof
```



eof块的内容会原封不动的保存