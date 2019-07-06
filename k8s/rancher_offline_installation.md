# rancher-server离线的高可用部署

本博客基于<https://www.cnrancher.com/docs/rancher/v2.x/cn/installation/air-gap-installation/ha/> 官方文档补充和完善，主要是博主按照官网文档操作时，遇到一些问题，这里作为重点说明

## 1，提前搭建后本地仓库

harbor离线仓库搭建，参见harbor官网和我的install-harbor 文档，注意的时，如果用https协议，需要启用域名的，自定义域名要配置/etc/hosts 把自定义域名加进去

##  2，把rancher所需的镜像导入到镜像库

可以利用脚本执行
```bash
./rancher-load-images.sh   -l  rancher-images.txt   -i  rancher-images.tar.gz   -r  仓库地址/library
或者其他项目名称，如提前创建一个rancher
./rancher-load-images.sh   -l rancher-images.txt  -i ./rancher-images.tar.gz -r 172.16.35.31:1180/rancher

```
上面脚本和所需镜像获取，参考官方文档
<https://www.cnrancher.com/docs/rancher/v2.x/cn/installation/air-gap-installation/ha/prepare-private-registry/>

## 3，二进制文件 kubectl  rke helm等copy指定位置

这些二进制文件，下载地址<https://www.cnrancher.com/docs/rancher/v2.x/cn/install-prepare/download/>

## 4，把镜像库的域名添加到其他机器hosts中、推送公钥等
* 需要注意如果仓库是https协议，自签名证书，需要把制作仓库证书生成的tls.crt  复制到其他docker主机上/etc/docker/certs.d/custom_domain/ca.crt 如：/etc/docker/certs.d/reg.czl.com/ca.crt 

## 5，制作自签名证书
```bash
./create_self-signed-cert.sh --ssl-domain=rancher.czl.com --ssl-trusted-ip=192.16.1.101
```
## 6，注意事项
剩下步骤按照官网文档：https://www.cnrancher.com/docs/rancher/v2.x/cn/installation/air-gap-installation/ha/install-kube/ 但是特别要注意下面事项

### 下面罗列一些注意事项

- yaml配置文件 模板：

```
nodes:
- address: 192.16.1.101            # node air gap network IP
  user: zhenglin
  role: [ "controlplane", "etcd", "worker" ]

private_registries:
- url: reg.czl.com/library #如果用的私有签名https协议的harbor，此处用域名，不用IP，另外如果rancher镜像导入到library项目，URL需要带上library；如果是http协议的直接填ip就可以
  user: admin
  password: "P@ssw0rd"
  is_default: true
```

- 安装Helm Server(Tiller) 这一步
  可以从https://hub.docker.com/r/hongxiaolu/tiller/tags/ 获取相应版本tiller
  也可以直接从 registry.cn-hangzhou.aliyuncs.com/google_containers/tiller:v2.13.1
  拉取，这个版本可以根据情况变化；然后再修改tag ,推送到私有仓库中

```
[root@Centos76 cert]# docker  tag    registry.cn-hangzhou.aliyuncs.com/google_containers/tiller:v2.13.1   reg.czl.com/library/rancher/tiller:v2.13.1  
[root@Centos76 cert]# docker  push  reg.czl.com/library/rancher/tiller:v2.13.1  
```


安装tiller
```
  helm init --skip-refresh \
      --service-account tiller \
      --tiller-image reg.czl.com/library/rancher/tiller:v2.13.1
```

-  先在chart解压目录下执行以下命令进行rancher安装，再来配置证书：

```
helm  install ../chart/rancher \
  --name rancher \
  --namespace cattle-system \
  --set hostname=rancher.czl.com \
  --set ingress.tls.source=secret \
  --set privateCA=true \
  --set rancherImage=reg.czl.com/library/rancher/rancher
```
*  一定要注意 其官方文档中```--set ingress.tls.source=secret``` 没有换行符 所以不能直接粘贴用,set rancherImage时，后面rancher的版本不要带上tag，因为默认他会自动寻找，以下是带上v2.2.2的tag 出现的异常
```
rancher-6f78c5bc69-g9djv   0/1     InvalidImageName   0          2m57s
kubectl   -n  cattle-system    edit  deploy  rancher 
image: reg.czl.com/library/rancher/rancher:v2.2.2:v2.2.2
```
可以看出 镜像的tag 被追加了
* 如果执行错了，可以用`helm delete --purge rancher` 删除后 重新执行

* 然后再到证书生成的目录 执行



```bash
kubectl -n cattle-system create \
secret tls tls-rancher-ingress \
 --cert=./tls.crt \
 --key=./tls.key


kubectl -n cattle-system \
create secret generic tls-ca \
--from-file=cacerts.pem
```

如下操作和在线安装文档一样

