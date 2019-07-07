# Rancher server 高可用集群在线安装

搭建集群前提是 服务器数量为大于1的奇数 本案例用三台机器，采用rke搭建k8s集群,然后利用helm在此集群中部署七层负载的rancher server。

## 通过 RKE 工具部署 Rancher Kubernetes 集群

### 部署前准备工作

- 首先安装前需要提前准备必要的工具软件，
  所需工具软件如下（只限于 Rancher 集群的三台节点使用）
  ✓ kubectl - Kubernetes 命令行工具。
  ✓ rke - Rancher Kubernetes Engine 用于构建 Kubernetes 集群。
  ✓ helm - Kubernetes 的包管理。
  可以通过：https://www.cnrancher.com/docs/rancher/v2.x/cn/install-prepar
  e/download/ 下载相关文件
  把事先下载完成的 rke 、helm 、kubectl 二进制文件 copy 到/usr/bin 目录下

  ```
  chmod 777 rke_linux-amd64 kubectl_linux-amd64 helm
  mv rke_linux-amd64 /usr/bin/rke
  mv kubectl_linux-amd64 /usr/bin/kubectl
  mv helm /usr/bin/helm
  ```

  Rancher 集群的三台主机分别修改本机的 hosts 文件

  ```
  cat <<EOF>> /etc/hosts
  ip1 hostname1
  ip2 hostname2
  ip3 hostname3
  EOF
  ```

  配置 Rancher 集群三台服务器之间的 ssh 免密登录
  

  配置 Rancher 集群三台服务器之间的 ssh 免密登录
  

  Rancher 集群的三台主机分别修改本机的 hosts 文件
  cat <<EOF>> /etc/hosts
  ip1 hostname1
  ip2 hostname2
  ip3 hostname3
  EOF
  配置 Rancher 集群三台服务器之间的 ssh 免密登录


### 生成 ssh 免密访问公钥
`ssh-keygen  -t rsa -C "备注信息"` 生成公私钥

将免密公钥推送集群内三台主机的 rancher账户(或其他可以使用docker的账号，centos不能使用root账号）

```
ssh-copy-id -i ~/.ssh/id_rsa.pub rancher@ip1
以此类推
三台机器都推送，包括自身
```

- 完成后验证是否可以和集群内三台主机完成免密登录

###  RKE 创建 Rancher Kubernetes 集群

创建 rancher-cluster.yml 文件，用于 rke 推送集群配置使用，配置如下：

```
nodes:
 - address: 10.128.119.47
 user: was
 role: [controlplane,worker,etcd]
 - address: 10.128.119.48
 user: was
 role: [controlplane,worker,etcd]
 - address: 10.128.119.49
 user: was
 role: [controlplane,worker,etcd]
services:
 etcd:
 snapshot: true
 creation: 6h

```

给提前下载好的 rke 工具配置可执行权限

chmod +x rke
### 执行集群部署操作
`./rke up --config rancher-cluster.yaml`

完成后（如果镜像拉取快，15分钟左右），正常显示：Finished building Kubernetes cluster successfully。

-  使用 Kubectl 验证集群健康状态

  ```
  mkdir -p ~/.kube
  cp kube_config_rancher-cluster.yml ~/.kube/config
  kubectl get nodes
  kubectl get cs
  ```

  至此k8s集群安装完成



## 通过 helm 工具在k8s集群部署 Rancher Server

###  配置 helm kubernetes 集群权限

安装 helm 工具包

```
tar -xvf helm-v2.13.1-linux-amd64.tar.gz
chmod +x linux-amd64/helm
cp linux-amd64/helm /usr/bin/
```

创建 helm ServiceAccount

```
kubectl -n kube-system create serviceaccount tiller
```



创建 ClusterRoleBinding 以授予 tiller 帐户对集群的访问权限

```
kubectl create clusterrolebinding tiller \
 --clusterrole cluster-admin --serviceaccount=kube-system:tiller
```

### 初始化 helm 并安装 tiller（helm server 端）

查看 helm client 端的版本号

```
helm version

helm init --skip-refresh --service-account tiller \
 --tiller-image \
 registry.cn-shanghai.aliyuncs.com/rancher/tiller:v2.13.1
```


 - tiller要和helm版本保持一致

### 准备安装 Rancher Server 相关证书

如果有自己官方证书，就用官方证书，如果没有证书，制作证书参考“证书申请与制作“章节

### 使用 helm 工具部署 Rancher Server

创建 namespaces
`kubectl create namespace cattle-system`


创建 tls 密文

```
kubectl -n cattle-system \
 create secret tls tls-rancher-ingress --cert=./tls.crt --key=./tls.key
```



创建 ca 密文

```
kubectl -n cattle-system \
 create secret generic tls-ca --from-file=cacerts.pem
```


添加 helm chart 仓库地址

```
helm repo add rancher-stable https://releases.rancher.com/server-charts/latest
```


- 查看是否添加成功

`helm repo list`
执行 Rancher Server 创建操作

```
helm install rancher-latest/rancher \
 --name rancher \
 --namespace cattle-system \
 --set hostname=自己的域名 \
 --set ingress.tls.source=secret \
 --set privateCA=true
```

- 注：自定义域名和证书申请的域名要保持一致

  ### 验证 Rancher Server 运行情况

查看是否部署成功

`helm list`

查看 pod

`kubectl get deployment -n cattle-system`

安装rancher server出错，可以用helm delete --purge rancher 删除后 重新执行

## 为Agent Pod添加主机别名

- 采用的是自定义域名而不是全局域名的话，需要在agent 里面配置自定义域名

- 如果是通过在/etc/hosts添加自定义域名方式指定的Rancher server访问URL，那么不管通过哪种方式(自定义、导入、Host驱动等)创建K8S集群，K8S集群运行起来之后，因为cattle-cluster-agent Pod和cattle-node-agent无法通过DNS记录找到Rancher server,最终导致无法通信。

- 可以通过给cattle-cluster-agent Pod和cattle-node-agent添加主机别名来解决；

在rancher UI界面，在system项目分别点击升级，进行编辑cattle-cluster-agent Pod和cattle-node-agent
点击显示高级选项；然后再网络里面 添加hosts别名，填写rancher server 服务器ip和自定义域名，完成后保存即可，待pod恢复后，状态会变成Active