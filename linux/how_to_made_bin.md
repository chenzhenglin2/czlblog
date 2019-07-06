# 如何制作二进制安装包

二进制安装包相当于把脚本和压缩包合成一个可执行的bin文件。

## 准备脚本和压缩文件

### 脚本准备

- 准备两个脚本

  - 起始脚本，用来和压缩包进行合并生成二进制文件的，命名为init.sh(也可以自主命名)

    ```bash
    #!/bin/bash
    export TARGET_DIR="$( cd "$( dirname "${BASH_SOURCE[0]}" )" && pwd )"
    #或export TARGET_DIR="$(cd $(dirname $0) && pwd )"
    AAABBB=`awk '/^__AAABBB_BELOW__/ {print NR + 1; exit 0; }' $0`
    echo "TARGET_DIR is: $TARGET_DIR"
    tail -n+$AAABBB $0 | tar xzv -C $TARGET_DIR &> /dev/null
    $TARGET_DIR/xx.sh | tee $TARGET_DIR/xx.log 
    exit 0
    __AAABBB_BELOW__
    
    ```

    代码注解：

    - 变量AAABBB是获取`__AAABBB_BELOW__` 下面一行的行数，然后从此行开始解压，解压完成后，执行解压出来的脚本，并把执行过程输出到指定日志。`__AAABBB_BELOW__`  下面一定要空一行

  

  

  - 功能脚本，放到压缩包中，待压缩包解压完成后，调用此脚本，实现目标功能的

    这个脚本根据生产实际情况来编写，可以是安装各种工具，各种功能的shell脚本。

### 压缩包准备

压缩包也是根据具体情况来准备，一般可以放各种rpm、安装包（可以是压缩包）、给起始脚本调用的shell文件。

所有文件准备完成后，采用` tar zcvf xxx.tgz xxx` 格式进行压缩

## 合成

```
cat init.sh xxx.tgz > install_$(date +%Y%m%d).bin
chmod a+x install_$(date +%Y%m%d).bin
```



## 测试和验证

找一个干净的linux环境，把合成后的bin文件copy过去，使用`./install_$(date +%Y%m%d).bin `进行安装验证，如果有问题，查看日志，然后进行排障。