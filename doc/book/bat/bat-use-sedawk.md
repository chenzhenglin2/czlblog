# 在bat脚本中巧用linux中的sed awk grep命令三巨头

## 场景描述

有批安卓设备，需要刷机后，生成一个系统序列号文件，推送到这台安卓设备；由于时间原因、刷机人员能力限制，要快速搞出一套程序，能特别简单的生成这个序列文件，并推送到设备上；

这种急需又简单的功能，可以考虑用bat脚本来快速实现，让操作人员只需双击就能完成上面需求。

序列号文件（名称为xlh.txt）：

```json
{"product_key":"11个字符","DeviceName":"16个字符","DeviceSecret":"32个字符"}
```

这种json格式的txt文档；

序列号列表（名称为aaa.txt)：

```
product_key   DeviceName    DeviceSecret
11个字符   16个字符   32个字符
11个字符   16个字符   32个字符
11个字符   16个字符   32个字符
11个字符   16个字符   32个字符
……
```

如果有shell基础的，一看这个需求，用sed 和 grep等命令很容易就编写出shell 脚本，但Windows中不支持这几个命令； 而且也不能让操作人员进行其他复杂操作。 所以就采用bat脚本实现，根据提供的序列号表生成新的序列号文件，然后推送到设备后，删除掉一行记录；

Windows系统安装git后gitbash中带有awk、sed、grep等工具打包放到脚本同级目录，然后cmd中使用awk  、sed 、grep命令看看，如果提示缺少dll库，把对应的库也copy进来。下面就是利用bat+sed等命令实现上述需求的脚本：

```
chcp 65001
@echo off

awk  'NR==1 {printf $0}'  aaa.txt  >  aaa.tmpp
for /f   %%i in ('awk '{print $1}' aaa.tmpp') do (set var=%%i)
del aaa.tmpp
if "%var%"=="DeviceName" (sed -i '1d' aaa.txt)

adb devices | awk 'END{printf NR}' > num.tmpp
for /f   %%j in ('type num.tmpp') do (set num=%%j)
del num.tmpp
::echo %num%
if %num% lss 3 ( 
	echo 没有发现设备，请检查设备是否连接到电脑
	pause
	exit
)

for %%a in (aaa.txt) do (
  if "%%~za" equ "0" (
  echo aaa.txt列表为空了
  pause
  exit
  )
)

awk  'NR==1 {printf $1}' aaa.txt  > device.tmpp
awk  'NR==1 {printf $2}' aaa.txt >  secret.tmpp
awk  'NR==1 {printf $3}' aaa.txt >  key.tmpp
for /f   %%x in ('type device.tmpp') do (set DeviceName=%%x)
for /f   %%y in ('type secret.tmpp') do (set DeviceSecret=%%y)
for /f   %%z in ('type key.tmpp') do (set ProductKey=%%z)
del *.tmpp
sed -ri "s|\"product_key\":\"[A-Za-z0-9]{11}\"|\"product_key\":\"%ProductKey%\"|;s|\"DeviceName\":\"[A-Za-z0-9]{16}\"|\"DeviceName\":\"%DeviceName%\"|;s|\"DeviceSecret\":\"[A-Za-z0-9]{32}\"|\"DeviceSecret\":\"%DeviceSecret%\"|"  xlh.txt	
TIMEOUT /T 1 /NOBREAK
sed -i '1d' aaa.txt		  
call push_xlh_and_install_flow.bat
pause
exit
```

上面脚本 ，为了保存变量都存到临时文件中，待取出变量值后，删除临时变量；把aaa.txt 列表文件抬头文件删除后，通过sed 命令生成新的的xlh.txt 序列号文件，并删除一行记录；调用push_xlh_and_install_flow.bat （这个脚本中实现了xlh.txt 推送到设备，并通过adb 等命令启动应用验证序列号等功能）；

对操作人员来说没有什么技术要求，只需把设备接入电脑后，双击这个脚本就能完成；没有接入设备，也会有相应的提示。





