# 如何在rancher平台宕机情况下，继续使用k8s集群

## 前提

- 首先保障rancher管理k8s集群的config文件还在，可以查看~/.kube/config 进行验证

```
kubectl  config    view 

kubectl config use（或者use-context ） xxx ；切换到不同配置项（master上的配置），来使用kubectl
#xxx是contexts的name值
```

## 验证：

```
kubectl  get  nodes 
kubectl  get  ns 
```

验证是否可以看到节点和命名空间

## 使用kubectl命令来操作集群

```
]$ kubectl   set image   deployment/reactor   reactor=myimage:2.7.1  -n  myns --dry-run(空跑一遍 进行验证）
]$ kubectl   set image   deployment/reactor   reactor=myimage:2.7.1  -n  myns

验证

]$ kubectl  get deployment/reactor   -n  myns  -o yaml
```



## 若rancher彻底不可恢复

待kubectl命令可以使用后，在新搭建rancher平台中，把此集群导入进去，导入完成后 要重建项目，把之前的命名空间移过去即可。

具体导入命令，参考rancher官网	