# 安装harbor说明

- 官方说明文档：https://github.com/goharbor/harbor/blob/release-1.7.0/docs/installation_guide.md
  选择在线还是离线根据情况，这里不作为操作重点；
- 离线安装包下载地址 https://github.com/goharbor/harbor/releases
- 如果采用https协议，并使用自签名的证书的话，可以使用https://github.com/xiaoluhong/server-chart/blob/v2.2.2/create_self-signed-cert.sh 脚本生成证书
## 在线安装http协议的harbor(默认使用此协议的，简单)

- 下载完成在线安装包解压后，在harbor目录，备份 docker-compose.yml后再修改  

```
container_name: nginx
 restart: always
 cap_drop:
   - ALL
 cap_add:
   - CHOWN
   - SETGID
   - SETUID
   - NET_BIND_SERVICE
 volumes:
   - ./common/config/nginx:/etc/nginx:z
 networks:
   - harbor
 dns_search: .
 ports:
   - 1180:80
   - 5555:443
   - 34443:4443
```
由于宿主机的80和443端口已经被用掉，只能映射成其他端口；如果是一台新机器 无需这么操作

### harbor.cfg备份后修改如下两个参数
```
hostname = 172.16.35.31:1180
ui_url_protocol = http
```
修改完成后执行 ./install.sh
然后就可以在浏览器中访问了：http://172.16.35.31:1180
在harbor/common目录有两个子目录，分别是：config     templates
其中templates是原始文件，config是执行install.sh 或prepare 后生成的，所以尽量修改templates目录下的文件；
如果要修改harbor的配置，按照官方文档“Managing Harbor's lifecycle” 来操作
执行install.sh 相当于
./prepare

docker-compose up -d
这两步，所以修改完成后 要么执行install.sh 要么执行这两步；

- 如果出现误操作，可以通过docker-compose  down  -v  再重新执行

### 如何使用http协议登录和推送镜像
最新版本docker 默认不允许使用http推送，所以我们要修改

* 在harbor宿主机和远程推送镜像的机器都增加如下配置
- vi /etc/docker/daemon.json
```
{ "insecure-registries":["172.16.35.31:1180"] }
或者直接在已有配置后面添加
{
  "registry-mirrors": [
    "https://registry.docker-cn.com"
  ],
  "insecure-registries":["172.16.35.31:1180"]
}
也可以使用0.0.0.0通配所有的http镜像仓库
"insecure-registries": ["0.0.0.0/0"]
```
* 修改harbor宿主机docker启动项配置/usr/lib/systemd/system/docker.service
  如果docker17.3版本，直接参照下面方式添加
```
ExecStart=/usr/bin/dockerd --insecure-registry=172.16.35.31:1180
```
如果docker是最新18.9或者以上版本，直接参照下面方式添加
```
ExecStart=/usr/bin/dockerd -H unix:///var/run/docker.sock --insecure-registry=172.16.35.31:1180
```
完成后，重启docker
```
systemctl    daemon-reload   
systemctl    restart  docker
```
验证参数：
```
ps aux|grep dockerd
root     106472 31.5  0.0 2150816 71576 ?       Ssl  17:06   0:02 /usr/bin/dockerd -H unix:///var/run/docker.sock --insecure-registry=172.16.35.31:1180
```

远程或者本地登录验证
```
docker  login  172.16.35.31:1180
Username: admin
Password:
Login Succeeded
```
* 若登录时提示``` net/http: request canceled (Client.Timeout exceeded while awaiting headers) ```
  这是代理搞的鬼，把代理/etc/systemd/system/docker.service.d/http-proxy.conf或者其他配置，里面
```
[Service]
Environment="HTTP_PROXY=http://10.xx.xx.83:3128"
Environment="NO_PROXY=localhost,127.0.0.0/8,10.42.*.*"
```
把登录时的ip  加入到NO_PROXY中，支持通配符

### 开放管理端口映射（http协议，可选操作）
 管理端口在 /lib/systemd/system/docker.service 文件中
 将其中第11行的 ExecStart=/usr/bin/dockerd 替换为：
 ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock -H tcp://0.0.0.0:7654
 （此处默认2375为主管理端口，unix:///var/run/docker.sock用于本地管理，7654是备用的端口）

将管理地址写入 /etc/profile
```echo 'export DOCKER_HOST=tcp://0.0.0.0:2375' >> /etc/profile```

管理接口以备Jenkins等工具调用使用。

## 离线安装https协议的harbor

###  生成证书
1，执行create_self-signed-cert.sh 带上相应参数生成所需自签名证书（可以-h,查看用法；如果有证书，忽略此步）
```bash
cat /etc/hosts
192.16.1.100 Centos76 reg.czl.com  具体名称和设置的自签名域名一致

./create_self-signed-cert.sh --ssl-domain=reg.czl.com --ssl-trusted-ip=192.16.1.100 
```
把生成的两个证书 copy到 /data/cert/
### 2，配置harbor
```
vi harbor.cfg
hostname = 域名 （不能填写IP，如果用IP的话，采用http协议方式安装harbor）
#自签名申请的域名如reg.czl.com 但采用了域名后，使用域名制作的证书，最好就填写域名避免出现 docker login ip 提示：certificate signed by unknown authority，linux终端用域名登录，浏览器端可以用域名也可以用https://ip登录，
#打tag和推送也必须用域名

ui_url_protocol = https

ssl_cert = /data/cert/tls.crt
ssl_cert_key = /data/cert/tls.key
#ssl_cert = /data/cert/server.crt
#ssl_cert_key = /data/cert/server.key

```
### 3，执行安装命令
```./install.sh```

### 4,如下操作和http协议的一样
但要注意，如果证书是自签名的 ，在其他机器登录此https协议的镜像库，还需要把tls.crt 推送过去
```bash
非仓库机器：
mkdir -p  /etc/docker/certs.d/reg.czl.com/
仓库机器把crt文件推送过去
scp  tls.crt   root@ip:/etc/docker/certs.d/reg.czl.com/ca.crt
如果是Ubuntu系统，root密码是随机的，可以分两步推送， 先推送给Ubuntu的常用账号，常用账号copy到指定目录
````

## 服务器断电重启后，发现连接不上harbor 如何优化
如果断电或者机器重启后，harbor不可用，十有八九就是容器没有自动起来，我们可以完善一下，这个组件自带的docker-compose.yml 文件
如下所示，在每个容器策略里面加上restart选项，设置为unless-stopped，新版1.7.5版本harbor 全部组件restart 选项都设为always  ,但还是需要手动停止部分组件后，再全部启动

### 注意事项
最好把harbor服务器的hostname或者别名加入到从harbor拉取镜像其他主机hosts文件中 
```
vim   /etc/hosts 
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
172.16.35.31 szly-manage
```

- **常见参数解释说明**

docker配置文件/etc/docker/daemon.json  或者启动脚本中 ExecStart参数
{"hosts": ["fd://", "tcp://0.0.0.0:2375"]}
fd：// 指自动进行unix socket它
和-H unix:///var/run/docker.sock 、-H unix:// 作用相同，只是不同docker版本，写法略有差异。高版本的docker都已经采用了  -H  这种形式参数

### 小贴士
直接登录界面后，新建项目，在项目里面点击推送镜像，可以弹出镜像推送方法

更详细配置和证书生成脚本可以参考rancher提供的文档
https://www.cnrancher.com/docs/rancher/v2.x/cn/installation/registry/single-node-installation/