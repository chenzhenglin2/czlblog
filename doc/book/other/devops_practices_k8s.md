---
typora-copy-images-to: images
typora-root-url: images
---



# 利用jenkins+harbor+k8s搭建起一套实时更新测试环境

## 场景描述

**有这么一场景，Java被测应用和Java探针一起制作成镜像后部署到k8s环境，当有新版探针发布时，要基于新版探针制作一个新的镜像，并在k8s集群中更换镜像。**

- 分析需求后，决定用两个jenkins   job来完成这个任务。

1. 第一个job，把原来编译打包探针job中增加触发下游项目和传递探针版本参数功能。

2. 第二个job，由第一个job触发；1，copy第一个脚本编译出来的探针文件，然后利用docker插件把最新探针文件连同被测应用镜像合成出 我们需要的镜像；2，推送到harbor仓库；3，最后在k8s集群更新镜像。



## 实现步骤

### job1的实现

-  插件准备："Environment Injector "  "Copy Artifact"   "Parameterized Trigger "  一些基础的插件这里不再说了；
-  原有job功能说明

​      我们探针源码有多个分支，有开发用的dev、测试用的test，以及正式版本uat-x.x.x(x为数字)；关于正式版呢，在测试通过后，会基于test分支创建出一个uat-x.x.x的正式版。这个时候人为构建job时，输入正式分支版本号，即可下拉这个版本源码，进行编译、和打包存档操作。

project 部分截图

拉取指定分支源码

![1563352085287](/1563352085287.png)

编译和存档，编译在execute shell模块实现编译，并对版本参数处理，去掉uat-,只保留数值。

![1563352263335](/1563352263335.png)

增加的两步

1，在execute  shell中 把截取后版本号保存到一个文件中

`echo  "SER_NO=${SER_NO}" > server_release_number.txt`

然后注入到环境变量

![1563352492070](/1563352492070.png)

2，触发下游项目并传递参数

增加构建后操作"trigger parameterized build on other projects",里面增加"Predefined parameters"

![1563352708618](/1563352708618.png)

### 构建工程2的实现

额外所需插件 Docker API

1，也增加构建参数，便于再没有接收到来自job1传递过来的参数时，可以手动输入

2，



