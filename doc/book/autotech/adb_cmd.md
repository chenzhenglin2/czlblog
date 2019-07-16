# adb常用命令

adb 直译是安卓调试桥，就电脑操作安卓设备用的；使用adb命令要注意如下两点：

- 安卓手机助手可能和自己adb占用同样端口，有冲突，这个需要手动解决；
- 安卓设备要调到开发模式。

##   Log抓取



`adb logcat –v time      > D:\xxx.txt `（-v为参数规定log的输出格式，-v time为按时间顺序输出log。箭头后面D:\为输出路径，可以修改路径，xxx.txt为log名称，可按自己需求确定名称）。

- Bugreport：

`adb bugreport >D:\xxx.txt` （箭头后解释同logcat）

- Trace：

`adb pull data/anr D:/XXX `(直接把log pull出来，adb pull 目标路径 本地路径)

- Kernel log：

`adb shell cat /proc/kmsg > name.txt `（此时默认存放在个人文件夹下，可以按个人需要修改存放路径，如>D:\name.txt,抓之前系统需root ---adb root—adb remount）

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
```



## shell终端可以输入的命令

```
adb shell     
logcat -v threadtime -n 1000 -r 512000 -f /mnt/sdcard/logcat &  //每个logcat存为512M
logcat -v threadtime -b events > /mnt/sdcard/events.txt >&1 &
logcat -v threadtime -b radio > /mnt/sdcard/radio.txt >&1 &
```

