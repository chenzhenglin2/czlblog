# centos7.6和Ubuntu19.04设置ip



## centos7任何版本都可以通过nmtui命令来设置ip，

但centos7.6ip的设置需要注意，在网卡配置文件/etc/sysconfig/network-scripts/ifcfg-eth0 ,需要配置如下两项 #网卡eth0名称因环境而已

```
ONBOOT=yes

IPV6_PRIVACY=no
```

一定要增加 IPV6_PRIVACY=no  ，centos7.6  默认采用IP6技术了



## Ubuntu19.4 配置静态IP

sudo vim  /etc/netplan/50-cloud-init.yaml   #文件名称因环境而异，但一定是*.yaml

```bash
network:

​    ethernets:

​        ens33:

​            dhcp4: no

​            addresses: [192.16.1.101/24, ]

​            gateway4: 192.16.1.2

​            nameservers:

​                    addresses: [192.16.1.2,8.8.8.8]

​    version: 2
```

完成后执行  sudo netplan apply

即可生效,博主参考了：<https://www.tecmint.com/configure-network-static-ip-address-in-ubuntu/>