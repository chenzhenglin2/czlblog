# Docker CE for CentOS 7.5/ubuntu19.04(centos7.5或以上版本)
## centos安装docker

### 安装yum⼯具
`sudo yum install -y yum-utils`

### 配置docker yum源
```
sudo yum-config-manager \
--add-repo \
https://download.docker.com/linux/centos/docker-ce.repo
sudo yum makecache fast
```
### 查询可⽤版本
```sudo yum list docker-ce --showduplicates | sort -r```

### 安装指定版本docker-ce-17.03.2.ce,
- 先安装docker-ce-selinux-17.03.2.ce，否则安装docker-ce会报错

  
```bash
sudo yum -y install https://download.docker.com/linux/centos/7/x86_64/stable/Packages/docker-ce-selinux-17.03.2.ce-1.el7.centos.noarch.rpm
```

- 2 安装docker-ce

```
sudo yum -y install docker-ce-17.03.2.ce-1.el7.centos
```

####  如果只下载安装包（和依赖库）：
```
yum install -y docker-ce-17.03.2.ce --downloadonly --downloaddir=./
```
### 安装最新版的docker
```
sudo yum -y install docker-ce
```
### centos（Redhat）离线安装
只需要下载对应版本RPM包 即可；
[阿里镜像源地址](https://mirrors.aliyun.com/docker-ce/linux/centos/7/x86_64/stable/Packages/)
如安装docker-18.09.5，下载对应的3个RPM包即可

```
containerd.io-1.2.5-3.1.el7.x86_64.rpm
docker-ce-18.09.5-3.el7.x86_64.rpm
docker-ce-cli-18.09.5-3.el7.x86_64.rpm
```


### 利用rancher提供的安装脚本在线安装docker

1，首先下载安装脚本
`curl  -OS https://releases.rancher.com/install-docker/18.09.sh  `

2，如果有连接外网VPN，可以使用
```curl https://releases.rancher.com/install-docker/18.09.sh  |sudo  sh ```直接安装，但这个不建议
 如果直接在线执行，缺点是无法修改脚本,失败率高，不建议,因为在centos安装时失败，并提示yum 源找不到可以通过修改一行代码，改用阿里的镜像源

 ```
centos|fedora|redhat|oraclelinux)
#yum_repo="https://download.docker.com/linux/centos/docker-ce.repo"
yum_repo="https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo"
 ```
3, 下载脚本后第一件事就是修改为阿里的镜像源
下载脚本 阿里镜像源地址：https://mirrors.aliyun.com/docker-ce/linux/
所以把
```
https://download.docker.com/linux
全部替换为：
https://mirrors.aliyun.com/docker-ce/linux
vi 或者vim 18.09.sh 
:%s/https:\/\/download.docker.com\/linux/https:\/\/mirrors.aliyun.com\/docker-ce\/linux/g
```
然后执行sh  18.09.sh 安装成功的

## Ubuntu安装docker


### Ubuntu执行rancher提供docker安装脚本
执行时报错：
```
+ sh -c apt-get install -y -q docker-ce=
Reading package lists...
Building dependency tree...
Reading state information...
E: Version '' for 'docker-ce' was not found
```
这个可能Ubuntu版本太高，需要手动来安装docker了
可以把下面命令执行完成后，再来执行rancher提供的docker安装脚本

```
sudo apt-get -y install apt-transport-https ca-certificates curl software-properties-common bash-completion
# step 2: 安装GPG证书
sudo curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
# Step 3: 写入软件源信息
sudo add-apt-repository "deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
# Step 4: 更新并安装 Docker-CE
sudo apt-get -y update
```

### ubuntu在线安装
```
sudo apt install docker.io(已过时)
或sudo  apt  install  docker-ce
```

### ubuntu离线安装
暂缺



*  rancher提供的docker安装脚本的GitHub：https://github.com/rancher/install-docker



## 改变docker的image存放⽬录（下面提供三种方式）、配置加速器等

### 修改配置文件，配置加速器

 中写入如下内容（如果文件不存在请新建该文件）
```
[root@yktapp1 dockersh]# cat   /etc/docker/daemon.json 
{
  "registry-mirrors": [
    "https://registry.docker-cn.com"
  ]
}
```



也可以使用阿里的镜像加速器和本地仓库配置`"https://8m0vweth.mirror.aliyuncs.com"`



### 使用内部仓库地址的配置



```json
[root@yktapp1 dockersh]# cat   /etc/docker/daemon.json 
{
"registry-mirrors": ["https://registry.docker-cn.com"],
"insecure-registries": ["0.0.0.0/0"]
}
```

`"insecure-registries": ["0.0.0.0/0"] `通配表示所有http协议仓库，如果只想通配某个http协议仓库，可以写成具体名称或者通配



```json
"insecure-registries":["10.251.66.44:5000"]

"insecure-registries":["10.251.26.0/24"]
```



- 参考harbor GitHub文档说明

  
```shell
If this private registry supports only HTTP or HTTPS with an unknown CA certificate, please add
--insecure-registry myregistrydomain.com to the daemon's start up arguments.

In the case of HTTPS, if you have access to the registry's CA certificate, simply place the CA certificate at /etc/docker/certs.d/myregistrydomain.com/ca.crt .
```



自签名https和不安全http都可以用这种方式解决，自签名的证书，也可以把证书copy到相应目录解决，无需添加insecure-registries选项，具体方法参见harbor的使用说明


- 注意，一定要保证该文件符合 json 规范，否则 Docker 将不能启动。

#### 1），指定docker存储位置，方法1（指定了overlay2，这种方式比overlay更高效，kernel较高的Linux默认就是这种）



```bash
cat > /etc/docker/daemon.json << EOF
{
"registry-mirrors": ["https://wgaccbzr.mirror.aliyuncs.com"],
"data-root": "/app/lib/docker"
}
{
"storage-driver": "overlay2",
"storage-opts":["overlay2.override_kernel_check=true"]
}
EOF
```



#### 2），把指定的盘符挂载给docker 

直接修改/etc/fstab中把一个逻辑卷挂到docker存储目录 /var/lib/docker中，但要确保原有的挂载没有东西，并取消之前的挂载关系， umout  -a   oldpath；可以通过下面查看原有的挂载和逻辑卷情况



```shell
[root@localhost ~]# lsblk
NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
fd0               2:0    1    4K  0 disk 
sda               8:0    0  100G  0 disk 
├─sda1            8:1    0    1G  0 part /boot
└─sda2            8:2    0   99G  0 part 
  ├─centos-root 253:0    0   50G  0 lvm  /
  ├─centos-swap 253:1    0  7.9G  0 lvm  [SWAP]
  └─centos-home 253:2    0 41.1G  0 lvm  /home
```



#### 3），软链的方式 （不推荐，作为挽救使用）



```shell
systemctl stop docker
mv /var/lib/docker /home/docker
ln -s /home/docker /var/lib/docker
systemctl start docker
```



## 安装 docker-compose



```bash
wget -c https://github.com/docker/compose/releases/download/1.22.0/docker-compose-`uname -s`-`uname -m`
cp docker-compose-Linux-x86_64 /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```




- 之后重新启动服务。

  
```bash
sudo systemctl daemon-reload
sudo systemctl start docker
sudo systemctl enable docker
```



- 注意最好配置完成好了 再启动docker，避免启动docker产生文件存储在默认的路径

### 测试安装是否成功
`docker ps `




## 非root用户操作docker
```bash
sudo gpasswd -a your-user  docker
or
sudo usermod -aG docker your-user
```

这两种方式 都是把非root用户添加到docker组

## Kernel性能调优

```bash
cat >> /etc/sysctl.conf<<EOF
net.ipv4.ip_forward=1
net.ipv4.neigh.default.gc_thresh1=4096
net.ipv4.neigh.default.gc_thresh2=6144
net.ipv4.neigh.default.gc_thresh3=8192
EOF
sysctl -p 
```

