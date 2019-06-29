# shell脚本中获取路径两种方法

## 第一种

```
DIR=$(cd $(dirname $0) && pwd )

echo  $DIR
```



## 第二种

```
DIR2=$(cd $(dirname "${BASH_SOURCE[0]}") && pwd )

echo  $DIR2
```



- 上面两种方法都可以获取到当前路径，但第二种方法只适用于含有bash命令的系统，若系统只有sh命令，建议采用第一种方式

