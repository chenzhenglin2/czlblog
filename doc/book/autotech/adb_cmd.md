# adb常用命令

adb 直译是安卓调试桥，就电脑操作安卓设备用的；使用adb命令要注意如下两点：

- 安卓手机助手可能和自己adb占用同样端口，有冲突，这个需要手动解决；
- 安卓设备要调到开发模式。

##   Log抓取



`adb logcat –v time      > D:\xxx.txt `（-v为参数规定log的输出格式，-v time为按时间顺序输出log。箭头后面D:\为输出路径，可以修改路径，xxx.txt为log名称，可按自己需求确定名称）。

如果想输出到文件的同时终端也显示相关信息，上面命令可以改进为：

```
adb logcat  -v time | tee D:\xxx.txt
```

进一步改进，如果想在logcat中过滤某个关键字 可以用

```
adb logcat  -v time | grep vehice
```

输出到一个指定文件中(shell类型终端)

```
adb logcat  -v time | grep vehice | tee D:\xxx.txt
```

但上面命令可能在cmd环境执行有问题，cmd对grep、tee命令支持不友好，可以用gitbash或其他bash终端来执行上面命令，或者改进命令为：

```
adb shell "logcat  -v time | grep vehice"  
```

这里不太好指定输出某一个文件了。如果想在cmd里面进行字段过滤，可以把grep替换为Windows支持的find 、 findstr 、Select-String 这几个关键字进行过滤；如：

```
adb logcat  -v time | Select-String "vehice"
```



- Bugreport：

`adb bugreport >D:\xxx.txt` （箭头后解释同logcat）

- Trace：

`adb pull data/anr D:/XXX `(直接把log pull出来，adb pull 目标路径 本地路径)

- Kernel log：

`adb shell cat /proc/kmsg > name.txt `（此时默认存放在个人文件夹下，可以按个人需要修改存放路径，如>D:\name.txt,抓之前系统需root ---adb root—adb remount）

- 查看某个app的版本号：

```shell
adb shell pm dump com.neolix_self_checking | findstr “versionName”
adb shell 后 ，pm dump com.neolix_self_checking | grep “versionName”   如果支持shell命令 可以合并成一个命令
```

- 如何通过命令启动APP

1，先查到active的名称

手动打开app ，通过日志查到MainActivity

`logcat  -v time  | grep  intent `

2,通过 am start 启动APP

```shell
adb shell "am start -n 包名/.activity.MainActivity"
```

MainActivity名称不是固定的，还有package_name.MainActivity 等多种命名方式。

- 如果电脑和安卓设备在同一局域网 ，可以无线连接

`adb  connect  ip:port`  port一般默认是5555

但是用完后  一定要断开，要不然其它电脑无法无线连接了；

`adb  disconnect`

- 进入到 Fastboot 模式：
  `adb reboot bootloader`

- 录屏和截图（真正应用时 ，可以直接用自己手机拍照和录制视频，灵活应用）

    ```
    adb shell screenrecord /sdcard/filename.mp4
    adb shell screencap -p /sdcard/sc.png
    ```

    

##   其他相关命令

```
adb devices-----查看设备是否连接成功，显示设备信息。

adb install ------安装apk文件

adb pull 目标路径 本地路径----获取系统中文件

adb push本地路径 目标路径---向系统中发送文件

adb shell -------进入系统的shell 模式

adb root --------获取管理员权限

adb remount---重新挂载系统分区，使系统分区重新可写，一般在adb root后使用

adb shell dumpsys package com.android.settings |grep version 查看应用版本，根据需要改变应用名称

adb shell getprop [ro.build.display.id](http://ro.build.display.id) 查看系统版本
adb wait-for-device logcat -v time  待设备连接成功后，输出log，wait-for-device一般放在adb命令后面，表示先等待设备连接，连接成功后，再执行后续命令；如adb wait-for-device root
也可以写成adb wait-for-device 
adb root

```



## shell终端可以输入的命令

```
adb shell     
logcat -v threadtime -n 1000 -r 512000 -f /mnt/sdcard/logcat &  //每个logcat存为512M
logcat -v threadtime -b events > /mnt/sdcard/events.txt >&1 &
logcat -v threadtime -b radio > /mnt/sdcard/radio.txt >&1 &
pm命令
pm install xxx.apk安装apk应用
pm uninstall app包名  卸载APP应用
pm list packages | grep nlix 查看包含某个关键字的包名

shell终端里面命令也可以用重定向 进行输入：
adb wait-for-device shell <<EOF
echo -n "Waiting for device to boot "
各种命令
甚至可以是shell代码如：
while cmp /data/local/tmp/zero /data/local/tmp/bootcomplete; do 
{
    echo -n "."
    sleep 1
    getprop dev.bootcomplete > /data/local/tmp/bootcomplete
}; done
echo "Booted."
exit
EOF
```

用adb 命令结合bat脚本，更方便的实现一些批量化操作。