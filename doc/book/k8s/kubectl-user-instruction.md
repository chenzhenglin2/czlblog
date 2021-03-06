

# kubectl 常用的命令总结

## kubectl 详细命令用法可以参考官网：

https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands

## 常用命令：

### 查看

- 只显示默认命名空间的pods

```kubectl get pods ```

- 显示所有空间的pod

```kubectl get pods  --all-namespaces```

- 显示指定空间的pod

```kubectl get pods  -o wide  --namespace apm```

其中--namespace  与-n 作用等同，后面接命名空间参数
`kubectl get deployment  -n  apm`
kubectl get pods,svc,rc  -n  apm  svc是services简称
这些命令都可以通过  kubectl get    --help 来查看帮助

### 删除
- 只能删除默认命名空间的deployment 

```kubectl  delete deployment  nginx  ```

- 删除指定空间的deployment/其他资源等

```bash
kubectl  delete TYPE RESOURCE -n NAMESPACE
具体如下：
kubectl  delete deployment shop-app -n test-shop  
kubectl  delete TYPE --all -n NAMESPACE
kubectl  delete all -n NAMESPACE
kubectl  delete all --all
```

### 创建、修改、打标签

- 使用yaml文件创建pod

```kubectl apply -f apptrace-receiver-deployment.yaml```
apply  和 create 命令都可以后跟yaml，创建所需资源,初次创建pod时可以互相替换使用；如果已有pod只是用于更新的话，又可以和replace相互替换使用；本着化繁就简的原则，create和replace都使用apply; 而且apply属于申明式语法，这个更加灵活，多次执行不会报错，只会更新改变的部分；像Jenkinsfile也已经从脚本语法向申明式转变。

- 使用kubectl命令把pod、卷、各种资源导出为yaml格式：

```bash
kubectl  get pods podA -n NAMEAPSCE-A -o yaml --export> padA.yaml
pod 可以换成其他申明式资源如卷、services等；如果不带上参数--export 目前没有发现有特别大的不同
kubectl  get all -n ns -o yaml --export> padA.yaml #导出某个命令空间所有资源
```
现在很多产品如rancher openshift，等；UI界面 直接可视化操作导出各种资源，掌握命令很多时候，可以事半功倍。-o更详细用法 下面有单独说明。

- 查看命名空间apm的collector服务详情

```kubectl  describe  service/apptrace-collector  --namespace apm 
--namespace 和-n  作用相同
```
- 查看pod日志

```kubectl  logs   podname  --namespace apm  (可以带上 -f 参数)```

- 为节点机apm-docker001打标签 zookeeper-node=apm-docker001,查看标签等；

```
为节点机打标签和查看
kubectl label nodes apm-docker001 zookeeper-node=apm-docker001
kubectl get nodes --show-labels
为命名空间打标签和查看
kubectl label namespace $your-namesapce istio-injection=enabled
kubectl  get namespaces  --show-labels
```
- 给名为foo的Pod添加label unhealthy=true

``` kubectl label pods foo unhealthy=true ```

- 查看某种类型字段下有哪些参数；

```
kubectl explain pods
kubectl explain Deployment
kubectl explain Deployment.spec
kubectl explain Deployment.spec.spec
kubectl explain Deployment.spec.template
kubectl explain Deployment.spec.template.spec

```

```
如查看Deployment.spec.template 可以有哪些参数
[root@k8s-master ~]# kubectl explain Deployment.spec.template
KIND:     Deployment
VERSION:  extensions/v1beta1

RESOURCE: template <Object>

DESCRIPTION:
     Template describes the pods that will be created.

     PodTemplateSpec describes the data a pod should have when created from a
     template

FIELDS:
   metadata     <Object>
     Standard object's metadata. More info:
     https://git.k8s.io/community/contributors/devel/api-conventions.md#metadata

   spec <Object>
     Specification of the desired behavior of the pod. More info:
     https://git.k8s.io/community/contributors/devel/api-conventions.md#spec-and-status
```
- 在pod内部，执行shell命令

```
kubectl exec podname printenv | ps aux | cat、ls 某个文件（如果pod不在默认空间，用-n 指定相应空间）
如：kubectl  exec   app-demo-68b4bd9759-sfpcf -n test-shop printenv

或者在kubectl exec podname -- 再跟shell命令 如：
kubectl exec -it spark-master-xksl -c spark-master -n spark  -- mkdir -p /usr/local/spark  
shell命令前，要加-- 号，不然shell命令中的参数，不能识别

kubectl  exec  后面只能是pod,目前还不支持deployment daemonnset等
```

### 特别说明

  创建和修改相对复杂些，可以直接在rancherUI 进行部署或者修改，修改还可以通过：kubectl edit deploy/xxx -n namespace,也可以把资源导出为yaml后，修改完成后，再次kubectl apply 部署一遍完成修改的目的。本人推荐通过rancherUI来部署和修改，这里就不对创建和修改做详细说明了；参考另一篇博客[rancher 中快速部署应用](deploy_app_in_rancher.md)

### "-"在kubectl中  用法说明

“-”它作为标准输入（池）非常灵活用法 ，和tar中用法非常类似

- yaml

  ```yaml
  - apiVersion: v1
    data:
      kubernetes.yml: |-
        - type: docker
          containers.ids:
          - "*"
          processors:
            - add_kubernetes_metadata:
                in_cluster: true
    kind: ConfigMap
  ```

  

- 命令

```bash
kubectl get po --all-namespaces --show-all --field-selector 'status.phase==Pending' -o json | kubectl delete -f -
```

```bash
curl --insecure -sfL https://your_domain/v3/import/f2gdpkvz42gqzm8vbrdpd99xgppjxgwct7wt86lzswwnf4p2d4vfd7.yaml | kubectl apply -f -
```



可以看出，“-”里面保存标准输入的内容。

## 巧用kubectl 帮助文件

如你只记得部分命令 get ,可以用
``` kubectl get   --help ```
同理
```kubectl create rolebinding ```
不知道后面接什么 也可以--help一下,记得关键字越多 带上后再使用help，如果只记得部分  就先help
如
```kubectl create  --help ```
这样create  所有类型的应用怎么创建 都有了
同样也可以直接  kubectl  --help 
这样kubectl  有哪些用法就显示出来，
我们要一级级的利用帮助  可能刚开始记住前面一个关键字，写完关键字 help一下  又有很多详细的用法 



## kubectl 命令规律总结
先看一组命令
```
kubectl  delete  sa  metricbeat  -n  efk
kubectl  get  sa  --all-namespaces
kubectl  delete  daemon-set  metricbeat  -n  efk
```
- 1.会发现，kubectl  不管get 、delete describe等操作 后面跟资源类型 如果sa(serviceaccout) deployment pod,然后是资源名称，如果没有资源名称，则删除、获取此类型所有的资源；最后限定某个命名空间，或者全部命名空间；这个限定命名空间 可以放在kubectl 后面，也可以放在所有参数后面；  也可以写成资源类型/资源名称；如

  `kubectl  delete  daemon-set/metricbeat  -n  efk`

- 2. -o 是指定输出格式
      输出格式	说明
      -o=custom-columns=<spec>	根据自定义列名进行输出，以逗号分隔
      -o=custom-colimns-file=<filename>	从文件中获取自定义列名进行输出
      -o=json	以JSON格式显示结果
      -o=jsonpath=<template>	输出jsonpath表达式定义的字段信息
      -o=jsonpath-file=<filename>	输出jsonpath表达式定义的字段信息，来源于文件
      -o=name	仅输出资源对象的名称
      -o=wide	输出额外信息。对于Pod，将输出Pod所在的Node名
      -o=yaml	以yaml格式显示结果

如下：
```
kubectl  get  sa   -n  efk  -o  yaml 
kubectl  get  sa  efk-elaticsearch -n  efk  -o  yaml  >xxx.yaml 
kubectl  get pod efk-elaticsearch-0 -n  efk  -o  wide
```



- 因为k8s 采用的是REST API接口，所有命令都最终会转换成curl  -X  PUT POS等形式,为什么不直接使用curl命令，因为需要一堆相关授权，rancher UI里面 在deploy或其他资源中，选择api查看 就可以查到，也可以点击右侧的edit编辑后 通过curl命令提交

```
API Request
cURL command line: 
curl -u "${CATTLE_ACCESS_KEY}:${CATTLE_SECRET_KEY}" \
-X PUT \
-H 'Accept: application/json' \
-H 'Content-Type: application/json' \
-d '{"annotations":{"cattle.io/timestamp":"", "cni.projectcalico.org/podIP":"10.42.1.44/32"}, "containers":[{"allowPrivilegeEscalation":false, "exitCode":null, "image":"172.16.35.31:1180/apm-images/gettoken:1.0", "imagePullPolicy":"IfNotPresent", "initContainer":false, "name":"genttoken", "ports":[{"containerPort":8001, "dnsName":"genttoken-nodeport", "kind":"NodePort", "name":"8001tcp301001", "protocol":"TCP", "sourcePort":30100, "type":"/v3/project/schemas/containerPort"}], "privileged":false, "procMount":"Default", "readOnly":false, "resources":{"type":"/v3/project/schemas/resourceRequirements"}, "restartCount":0, "runAsNonRoot":false, "state":"running", "stdin":true, "stdinOnce":false, "terminationMessagePath":"/dev/termination-log", "terminationMessagePolicy":"  等等
```
修改完成后 可以点击send  request来提交



## k8s所有资源类型和别名（缩写）

```bash
~]# kubectl  api-resources  
NAME                              SHORTNAMES   APIGROUP                       NAMESPACED   KIND
bindings                                                                      true         Binding
componentstatuses                 cs                                          false        ComponentStatus
configmaps                        cm                                          true         ConfigMap
endpoints                         ep                                          true         Endpoints
events                            ev                                          true         Event
limitranges                       limits                                      true         LimitRange
namespaces                        ns                                          false        Namespace
nodes                             no                                          false        Node
persistentvolumeclaims            pvc                                         true         PersistentVolumeClaim
persistentvolumes                 pv                                          false        PersistentVolume
pods                              po                                          true         Pod
podtemplates                                                                  true         PodTemplate
replicationcontrollers            rc                                          true         ReplicationController
resourcequotas                    quota                                       true         ResourceQuota
secrets                                                                       true         Secret
serviceaccounts                   sa                                          true         ServiceAccount
services                          svc                                         true         Service
mutatingwebhookconfigurations                  admissionregistration.k8s.io   false        MutatingWebhookConfiguration
validatingwebhookconfigurations                admissionregistration.k8s.io   false        ValidatingWebhookConfiguration
customresourcedefinitions         crd,crds     apiextensions.k8s.io           false        CustomResourceDefinition
apiservices                                    apiregistration.k8s.io         false        APIService
controllerrevisions                            apps                           true         ControllerRevision
daemonsets                        ds           apps                           true         DaemonSet
deployments                       deploy       apps                           true         Deployment
replicasets                       rs           apps                           true         ReplicaSet
statefulsets                      sts          apps                           true         StatefulSet
tokenreviews                                   authentication.k8s.io          false        TokenReview
localsubjectaccessreviews                      authorization.k8s.io           true         LocalSubjectAccessReview
selfsubjectaccessreviews                       authorization.k8s.io           false        SelfSubjectAccessReview
selfsubjectrulesreviews                        authorization.k8s.io           false        SelfSubjectRulesReview
subjectaccessreviews                           authorization.k8s.io           false        SubjectAccessReview
horizontalpodautoscalers          hpa          autoscaling                    true         HorizontalPodAutoscaler
cronjobs                          cj           batch                          true         CronJob
jobs                                           batch                          true         Job
certificatesigningrequests        csr          certificates.k8s.io            false        CertificateSigningRequest
clusterauthtokens                              cluster.cattle.io              true         ClusterAuthToken
clusteruserattributes                          cluster.cattle.io              true         ClusterUserAttribute
leases                                         coordination.k8s.io            true         Lease
bgpconfigurations                              crd.projectcalico.org          false        BGPConfiguration
clusterinformations                            crd.projectcalico.org          false        ClusterInformation
felixconfigurations                            crd.projectcalico.org          false        FelixConfiguration
globalnetworkpolicies                          crd.projectcalico.org          false        GlobalNetworkPolicy
globalnetworksets                              crd.projectcalico.org          false        GlobalNetworkSet
hostendpoints                                  crd.projectcalico.org          false        HostEndpoint
ippools                                        crd.projectcalico.org          false        IPPool
networkpolicies                                crd.projectcalico.org          true         NetworkPolicy
events                            ev           events.k8s.io                  true         Event
daemonsets                        ds           extensions                     true         DaemonSet
deployments                       deploy       extensions                     true         Deployment
ingresses                         ing          extensions                     true         Ingress
networkpolicies                   netpol       extensions                     true         NetworkPolicy
podsecuritypolicies               psp          extensions                     false        PodSecurityPolicy
replicasets                       rs           extensions                     true         ReplicaSet
nodes                                          metrics.k8s.io                 false        NodeMetrics
pods                                           metrics.k8s.io                 true         PodMetrics
alertmanagers                                  monitoring.coreos.com          true         Alertmanager
prometheuses                                   monitoring.coreos.com          true         Prometheus
prometheusrules                                monitoring.coreos.com          true         PrometheusRule
servicemonitors                                monitoring.coreos.com          true         ServiceMonitor
networkpolicies                   netpol       networking.k8s.io              true         NetworkPolicy
poddisruptionbudgets              pdb          policy                         true         PodDisruptionBudget
podsecuritypolicies               psp          policy                         false        PodSecurityPolicy
clusterrolebindings                            rbac.authorization.k8s.io      false        ClusterRoleBinding
clusterroles                                   rbac.authorization.k8s.io      false        ClusterRole
rolebindings                                   rbac.authorization.k8s.io      true         RoleBinding
roles                                          rbac.authorization.k8s.io      true         Role
priorityclasses                   pc           scheduling.k8s.io              false        PriorityClass
storageclasses                    sc           storage.k8s.io                 false        StorageClass
volumeattachments                              storage.k8s.io                 false        VolumeAttachment
```

k8s的资源类型，其中常用的资源一般有SHORTNAMES，在输入命令时，输入这种简称，无疑能提高效率。

kubectl get cs，要比kubectl get  componentstatuses 书写快的多，更容易记住。

## 更多用法可以参照官网或者国内翻译的博客

https://blog.csdn.net/xingwangc2014/article/details/51204224

