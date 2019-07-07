# 安卓设备monkey测试

## 1

```
adb shell
```



## 2

```
logcat -v threadtime -n 1000 -r 512000 -f /mnt/sdcard/logcat &          
```

​         

这个命令限制每个logcat文件大小为500MB以内，便于查看；保存位置是手机存储中

### 3

```bash
monkey -p com.xxx.auto -s 1000 --ignore-crashes --ignore-timeouts --ignore-security-exceptions  --pct-trackball 0 --pct-nav 0 --pct-majornav 0 --pct-anyevent 0  -v -v -v --throttle 500 1200000000 > /mnt/sdcard/monkey.log 2>&1 &    
```

​      com.xxx.auto为APP的包名，这个可以通过开启手机log，然后操作此app，从日志中获取到这个包名。         

 



