# getopt 与 getopts用法详解

我们经常使用脚本 后面跟参数这种用法，这个时候使用getopt/getopts再合适不过了；下面就来详细说明 getopt （系统外部用法，后来增加的）与 getopts（内部，不支持长选项 只能是单个字符的短选项）的用法

## 先看一段代码



```
#!/bin/bash
set -e
set -o pipefail

cmd=$(basename $0)
defaultHost="127.0.0.1"
defaultPort="2222"
defaultFusion="http://$defaultHost:3333"
defaultUser="myuser"
defaultPassword="123456"

host=$defaultHost
port=$defaultPort
fusion=$defaultHost
user=$defaultUser
password=$defaultPassword
if which getopt &>/dev/null; then
    optExist="true"
fi

Usage() {
   echo "帮助说明"
   echo "这个脚本如何使用，如参数 a f  h 后面跟什么值，或者长参数appid fusion help 后面跟什么值"
}

if [[ "${optExist}"x = "true"x ]]; then
    ARGS=$(getopt -o "a:f:h" -l "appid:,fusion:,help" -n "${cmd}" -- "$@")

eval set -- "${ARGS}"

while true; do
    case "${1}" in
        -a|--appid)
        shift ;
        if [[ -n "${1}" ]]; then
            appid="${1}"
            shift ;
        fi
        ;;
        -f|--fusion)
        shift ;
        if [[ -n "${1}" ]]; then
            fusion=${1}
            shift ;
        fi
        ;;
        -h|--help)
        Usage
        exit 0
        ;;
        --)
        shift ;
        break ;
        ;;
        *)
        Usage
        exit 0
        ;;
    esac
done

else
    while getopts a:f:h opt; do
        case $opt in
            a)
            appid=$OPTARG
            ;;
            f)
            fusion=$OPTARG
            ;;
            h)
            Usage
            exit 0
            ;;
            ?)
            Usage
            exit 1
            ;;
        esac
    done
fi

if ! [[ $appid =~ [0-9a-fA-F]{24} ]]; then
    echo -e "\033[1;41;37mInvalid appid!\033[0m\n"
    Usage
    exit 1
fi

if ! [[ $fusion =~ ^http:// ]]; then
    fusion="http://${fusion}:3000"
fi

echo -e "\033[32mSending request...\033[0m"
curl -H "Content-Type:application/json" -X PUT --data "{\"app_id\": \"$appid\"}"  "$fusion/api/v1/app/indexes"
echo -e "\033[32mFinished!\033[0m"
exit 0
```

## 代码详解：

先看getopt 与getopts 帮助文档

```
#[root@gitbook ~]# getopt  --help
#Usage:
# getopt optstring parameters
# getopt [options] [--] optstring parameters
# getopt [options] -o|--options optstring [options] [--] parameters

#Options:
# -a, --alternative            Allow long options starting with single -
# -h, --help                   This small usage guide
# -l, --longoptions <longopts> Long options to be recognized
# -n, --name <progname>        The name under which errors are reported
# -o, --options <optstring>    Short options to be recognized
# -q, --quiet                  Disable error reporting by getopt(3)
# -Q, --quiet-output           No normal output
# -s, --shell <shell>          Set shell quoting conventions
# -T, --test                   Test for getopt(1) version
# -u, --unquoted               Do not quote the output
# -V, --version                Output version information
……
#[root@gitbook ~]# getopts
# getopts: usage: getopts optstring name [arg]

```

 getopts a:f:h opt  表示a f  h 必须取值，使用选项取值时，必须使用变量OPTARG保存该值



- `basename $0`值显示当前脚本或命令的名字
-  shift 命令每执行一次，变量的个数($#)减一，而变量值提前一位
   比如shift 3表示原来的$4现在变成$1，原来的$5现在变成$2等等，原来的$1、$2、$3丢弃，$0不移动。不带参数的shift命令相当于shift 1
-  $#: 参数的个数

- `$*`: 参数列表  所有的参数 作为一个整体 如 ./xx.sh 1 2 3, `$*`值为“1 2 3”

- `$@`: 参数列表 所有参数 逐个输出 如 ./xx.sh 1 2 3  `$@` 值为"1" "2" "3"

- `set -- "${ARGS}"`  相当于重置了 执行脚本 后跟参数，把执行脚本后跟参数‘$@’  变成了 ${ARGS}

  

### 代码大意

- 先判断是否有getopt命令，如果有就用这个命令  没有的话 采用getopts命令，这个命令简单，不再做特殊说明；

-  getopt 判断体中  首先判断 第一个参数如果是 -a|--appid，则shift 后移一位 也是就是-a 参数后面的值（如果有的话）保存起来； shift 执行一次，然后$1(${1}写法也可以)就变成第三个参数了，  如果 -f|--fusion是第一个参数，操作效果和上面一样；如果第一个参数是 -h|--help    的话调用Usage方法并执行exit 0  退出；

-  如果第一个参数是 --  则shift 一下，break 跳出整个循环，（continue一般是 跳出当前循环，然后开始新的循环）； 如果第一个参数是为其他(?通配其他内容)调用Usage方法并执行exit 1退出

  其他代码较简单 不做讲解说明





