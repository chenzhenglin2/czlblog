# 制作自签名证书



## 1，手动制作自签名证书（NGINX用）

```
 openssl req -newkey rsa:2048 -nodes -keyout tls.key -x509 -days 3650 -out tls.pem -subj /C=CN/ST=BJ/L=CY/O=DCLINGCLOUD/OU=APM/CN=apptrace/emailAddress=ca@dclingcloud.cc
```



- 注：-keyout 和 -out  可以修改为输出路径+文件名称，名称可以自定义

字段说明：

C=CN          // 国家代号，中国输入CN

ST=BJ         // 州（省）名

L=CY          // 所在地市的名称

O=DCLINGCLOUD // 组织或者公司名称

OU=APM        // 部门名称

CN=apptrace   // 通用名，可以是服务器域控名称，或者个人的名字

emailAddress=ca@dclingcloud.cc  // 管理邮箱名



## 2，利用脚本制作所需证书（可以用在harbor和kubernetes、rancher部署上）

脚本下载地址（rancher中国提供）

<https://github.com/xiaoluhong/server-chart/blob/v2.2.4/create_self-signed-cert.sh>

脚本使用示例

```
./create_self-signed-cert.sh --ssl-domain=www.test.com --ssl-trusted-domain=www.test2.com \
--ssl-trusted-ip=1.1.1.1,2.2.2.2,3.3.3.3 --ssl-size=2048 --ssl-date=3650
```

具体用法可以参考脚本下载链接 