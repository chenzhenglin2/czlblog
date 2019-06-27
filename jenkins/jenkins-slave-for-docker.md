**如果用docker 容器编译程序 有两种方案可供选择**

- **1，激活镜像作为slave编译**

- 采用Jenkins提供的jnlp-slave 或ssh-slave  标准镜像二次封装，或者初始镜像，然后通过label 选择镜像后进行编译；

  这种编译的原理是：Jenkins通过标签选择相关docker镜像，并激活成容器，把此容器当做slave（节点机）使用；

​     Jenkins提供的镜像地址：<https://hub.docker.com/u/jenkinsci>

​     jnlp-slave 和ssh-slave 镜像都能激活作为slave节点使用，区别是采用不同协议连接到容器内部：ssh和jnlp；

使用这种方式需要额外在Jenkins里面配置，插件里面安装docker插件，然后配置Docker Host URL,来找到可以使用的docker，如果Jenkins和docker在同一台服务器可以直接填写为：unix:///var/run/docker.sock ，如果不在同一台机器要填写docker所在的机器ip：tcp://ip:2375，并且要在docker所在机器的 /etc/docker/daemon.json 里面添加2375端口

```
    "hosts": [

​        "tcp://0.0.0.0:2375",

​        "unix:///var/run/docker.sock"

​    ]
```


具体怎么调用这里不作为重点，这里重点说明的方案的选择，可以参考

<https://blog.csdn.net/qq_31977125/article/details/82999872>   这位博主有详细的操作步骤

下面是重点说明编译方式

- **2，直接docker run编译**

把源码下载到宿主机，通过-v 挂载到容器中，然后指定入口命令编译此目录，编译完成后 销毁容器



```
docker run --rm -v `pwd`:/mypro -w /mypro nodeshift/centos7-s2i-nodejs:10.16.0 /bin/bash -c "npm install"

 docker run --rm -v `pwd`:/opt/mypro -w /opt/mypro goenv-centos:test7  /bin/bash -c "GOPROXY=https://goproxy.io /usr/local/go/bin/go  build"
```

上面两个是例子，goenv-centos:test7是自制作的镜像

注意：提前制作好镜像，镜像里面把各种编译依赖放进去，制作镜像有两种方式

​         激活一个基础镜像，编译下载各种依赖库，能正式编译后，commit容器为镜像，这种方式不推荐

​        建议把操作步骤整理起来 编写到dockerfile中   

**实战案例**

基于centos7.6 镜像制作出可以编译go/rust程序的镜像（生产中最好一个镜像只编译一种语言程序，并且是基于此语言的镜像）

```bash
Dockerfile

FROM centos:7.6.1810
WORKDIR /opt
ADD https://github.com/facebook/zstd/releases/download/v1.4.0/zstd-1.4.0.tar.gz ./
ADD https://github.com/edenhill/librdkafka/archive/master.zip  ./
ADD https://studygolang.com/dl/golang/go1.12.5.linux-amd64.tar.gz  ./
COPY expect.sh ./
RUN mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.bak && \
    curl -s  http://mirrors.aliyun.com/repo/Centos-7.repo  -o  /etc/yum.repos.d/CentOS-Base.repo && \
    curl -s  http://mirrors.aliyun.com/repo/epel-7.repo -o  /etc/yum.repos.d/epel-7.repo && \
    yum repolist && \
    yum -y install wget make expect gcc gcc-c++ unzip openssl-devel  clang-devel libpcap-devel perl.x86_64 \
    which.x86_64 git && \
    curl  -sSf  https://sh.rustup.rs -o rustup.sh && chmod  a+x rustup.sh && \
    chmod  a+x expect.sh && /bin/bash ./expect.sh && \
    source $HOME/.cargo/env && \
    tar xzf zstd-1.4.0.tar.gz  && \
    tar xzf go1.12.5.linux-amd64.tar.gz -C /usr/local  && \
    unzip  master.zip  && \
    cd zstd-1.4.0 && CFLAGS="-O3 -fPIC" make install && \
    cd ../librdkafka-master && ./configure --prefix=/usr && make && make  install && \
    cp  -rf /usr/lib/librdkafka* /usr/lib/pkgconfig  /usr/lib64 && \
    chmod a+x -R /usr/local/go && \
    echo -e 'export GOPROXY=https://goproxy.io' >> /etc/profile && \
    echo -e 'export GOROOT=/usr/local/go' >> /etc/profile && \
    echo -e 'export GOPATH=/opt/mypro' >> /etc/profile && \
    echo -e 'export export PATH=$GOROOT/bin:/root/.cargo/bin:$PATH' >> /etc/profile && \
    echo -e 'export LIBRARY_PATH=$LIBRARY_PATH:/usr/lib64/llvm' >> /etc/profile && \
    echo -e 'export LD_LIBRARY_PATH=/usr/local/lib/:${LD_LIBRARY_PATH}' >> /etc/profile && \
    source /etc/profile && \
    yum clean all && \
    rm -rf /opt/* && \
    mkdir -p /opt/mypro
COPY ustc-config /root/.cargo/config
```

**ustc-config**

```
[source.crates-io]
registry = "https://github.com/rust-lang/crates.io-index"
replace-with = 'ustc'
[source.ustc]
registry = "git://mirrors.ustc.edu.cn/crates.io-index"
```

**expect.sh**



```bash
#!/usr/bash
expect<<-END
  spawn bash rustup.sh
  expect "1)&quot;Proceed&quot;with&quot;installation&quot;(default)"
  send "1\n"
set timeout -1
expect eof
exit
END
```



把Dockerfile，和expect.sh、ustc-config放在同一目录，然后在此目录执行build命令：

```
docker build  -t 172.16.35.31:1180/apm-images/centos7-goenv:1:1  .
```

- go程序编译：

```
docker run --rm -v `pwd`:/opt/mypro -w /opt/mypro  172.16.35.31:1180/apm-images/centos7-goenv:1  /bin/bash -c "GOPROXY=https://goproxy.io /usr/local/go/bin/go  build"
```

- rust程序编译

```
docker run --rm -v `pwd`:/opt/mypro -w /opt/mypro 172.16.35.31:1180/apm-images/centos7-goenv:1  /bin/bash -c "/root/.cargo/bin/cargo build --release"
```

这种方式编译最大优点，不管是开发、运维还是测试都无需搭建编译环境（有时候搭建一套编译环境是费时费力的事情，而且移植性差），直接从docker仓库拉取此镜像后就能编译；