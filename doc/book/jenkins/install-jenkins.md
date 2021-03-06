# 利用docker image快速搭建Jenkins环境  

关于安装部署Jenkins，网上一大堆资料，这里不做详细说明了；可以下载Jenkins的war包直接放到tomcat（其他Java容器也行）通过ip:8080/jenkins即可访问，也可能通过 java –jar Jenkins.war来安装或者带上--ajp13Port=-1 --httpPort=8081参数指定端口就行；

这里重点说的是采用docker化部署，这种方案更快捷和灵活：

```
docker run \
  --name myjenkins \
  --cpus=4 \
  --restart=unless-stopped \
  -u root \
  -d \
  -p 88:8080 \
  -p 50002:50000 \
  -v /home/jenkinsci/jenkins:/var/jenkins_home \
  -v /opt/jenkins-bak-file:/opt/jenkins-bak-file \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /etc/localtime:/etc/localtime:ro \
  -v /etc/timezone:/etc/timezone:ro \
  -v $HOME/.ssh:/root/.ssh:ro \
  jenkinsci/blueocean
```

* /etc/timezone  如果没有可以创建一个:echo Asia/Shanghai >/etc/timezone 这个 一定要设置，要不然就是GMT时区了；
* /opt/jenkins-bak-file目录是为以后Jenkins自身备份预留的目录；
* 把.ssh挂载进去，让容器使用宿主机的私钥；
* 如果使用jenkins:2.60.3镜像的话，宿主机的/home/Jenkins 目录一定要sudo chown -R 1000 处理一下；但不推荐这种镜像，已经不支持blueocean了；

如果容器中使用默认的jenkins账户，需要把宿主机的/home/jenkinsci/jenkins目录做一下`sudo chown 1000  -R`处理，另外宿主机操作docker账户的$HOME/.ssh目录挂载到容器/var/jenkins_home/.ssh下，这样容器就可以使用宿主机的公私钥；更保险的方法是直接进入jenkins容器，生成公私钥，因为公私钥所在目录/var/jenkins_home挂载到宿主机，不用担心重启后找不到的问题；