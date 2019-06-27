# "-"在tar命令中的巧用

### **首先来看示例：**



`tar -cvf - /home | tar -xvf -`



前面把压缩结果存到-，后面通过管道 | 把存到-中的文件解压，如果纯粹看这个，觉得这不瞎折腾么，下面从实战来说明使用它的好处



### **实战案例1：**

海量小文件传输方法



```
接收机：nc -l 8888 | tar xzf - -C /dest-dir 

发送机：tar czf - /source-dir/  | nc 接收机ip 8888
```



在 GNU 指令中，如果单独使用 - 符号，不加任何该加的文件名称时，代表"标准输入/输出"的意思,上面命令把结果输入到-,然后再解压-; 接收机可以带上参数v,如 xzvf（便于可视化）；如果发送机压缩命令带有z接收机也必须带上参数z  另外8888或其他端口，一定要放开，或者关闭防火墙；通过这种方式传递文件，相等于把先把文件压缩、然后通过scp 传输、再解压这几步合并成两步，并且省去了等待压缩和解压的时间。



### **实战案例2：**



`find /directory -type f -name "mypattern" | tar -cf archive.tar -T -`



找到匹配的文件后，直接压缩



### **实战案例3 docker cp 命令**

docker cp命令中用法

```
]# docker  cp  --help

Usage:  docker cp [OPTIONS] CONTAINER:SRC_PATH DEST_PATH|-

​        docker cp [OPTIONS] SRC_PATH|- CONTAINER:DEST_PATH
```



官网解释：

```
Use ‘-‘ as the source to read a tar archive from stdin and extract it to a directory destination in a container. Use ‘-‘ as the destination to stream a tar archive of a container source to stdout.
```



所以我们可以



```
docker  cp cfcc20077ad1:/opt/aa.tgz -  |  tar xvf  -  -C ./xx

tar  czf   -  anaconda-ks.cfg   initial-setup-ks.cfg   | docker cp  -  cfcc20077ad1:/opt/mydir
```



tar成对使用，前面一个是stdin  后面一个是stdout  ,  最终文件不会改变，-里面存的是压缩包，传输完成后还是压缩包，如果是文件传输完成后还是文件 ，只不过tar 传输比较快

