# 如何去掉行首行尾的空格

## 通过sed替换方法去掉行首或行尾的空格

```bash
$ echo   -e  "Hello  Word    "  | sed 's#\s*##;s#\s*$##'
Hello  Word
```

- #可以与/互换，避免混淆这里统一用#来表示分隔符
- \s匹配任何空白字符，包括空格、制表符、换页符等等,等价于` [ \f\n\r\t\v]`
- \S匹配任何非空白字符。等价于  `[^ \f\n\r\t\v]`。

## 非常规方法

在GitHub浏览代码时，发现<https://github.com/dylanaraps/pure-bash-bible>一个特别好的bash代码应用实例，上面截取字符串方法如下：

- 去掉行首和行尾的空格

```bash
trim_string() {
    # Usage: trim_string "   example   string    "
    : "${1#"${1%%[![:space:]]*}"}"
    : "${_%"${_##*[![:space:]]}"}"
    printf '%s\n' "$_"
}
```

结果：

```bash
$ trim_string "    Hello,  World    "
Hello,  World
```

代码写的很简洁，注释太少；这里做一下注解 ，方便自己理解和以后使用，可以给大家提供一个参考 

-  首先看方法体格式

method(){

: 参数1

: 参数2

}

它里面有注解：`The : built-in is used in place of a temporary variable.`  用内置函数冒号替代临时变量 ；和`${parameter:word}`  这种用法类似，把处理后得到的值存储到冒号中了； 这种格式方法，值得借鉴。

- "$_" 是保存之前执行命令 最后一个参数
- 字符串截取可以参考 https://www.cnblogs.com/xwdreamer/p/3823463.html
- 这是假设一个字符串为： "两个空格Hello空格Word三个空格"
- 字符串截取知识掌握后，再来逐步分析`${1#"${1%%[![:space:]]*}"}` 作用  ，${1}表示存储所有字符串的变量，然后分析“${1%%[![:space:]]\*}”这个值， 他表示截从右（最后一个非空格）往左截取，非空格开始到行首，即其值为两个空格；然后变成${1#"两个空格"}，再次用#，以两个空格（值）为起始，往后截取一次， 最后的值就是 "Hello空格Word三个空格"
- `${_%"${_##*[![:space:]]}"}`,${_}表示存储之前操作后得到的变量；`##*[![:space:]]` 表示最后一个非空格到结尾，即值为“三个空格”，然后从后往前把三个空格截取掉。最终形成去掉行首和行尾的空格了。

## 发散

[![:space:]]能不能像sed中用到的正则换成\S，尝试了一次，没有成功，然后又替换成`[^ \f\n\r\t\v]` 这种非简写的格式，最后也可以实现这个效果；printf  格式输出换成echo也可以 。



```
trim_stringw() {
    # Usage: trim_string2 "   example   string    "
    : "${1#"${1%%[^ \f\n\r\t\v]*}"}"
    : "${_%"${_##*[^ \f\n\r\t\v]}"}"
    echo  "$_"
}
```



## 去掉所有多余空格，中间的空格只保留一个空格

```bash
trim_all() {
    # Usage: trim_all "   example   string    "
    set -f
    set -- $*
    printf '%s\n' "$*"
    set +f
}
```

这个方法比较好理解，通过set -- $*  去掉了多余的空格，只保留一个空格，set -f /+f 取消/增加通配符使用后，方便格式化；



​	