# 一些常用nginx小知识汇总

- **nginx自动调整进程数（调整到和核数同样大小）**

`worker_processes  auto;`

- **调整客户端最大body大小**

`client_max_body_size 20m;`

- **包含其他配置（这样就更便于模块化配置）**

`include /etc/nginx/conf.d/*.conf;`	

- **某一个访问（80或其他）端口，强制跳转到https(443)协议端口上**

`rewrite (.*) https://$server_name$request_uri redirect;`

- **如果是非http协议的 负载均衡，可以直接使用stream，然后里面包含upstream（轮询）模块**

```
stream {
  upstream dns {
      server 127.0.0.1:3002;
      #server otherip:3002 weight=5;  此处用来负载均衡指派
    
  }
```

保留客户端真实ip

客户访问服务器经过nginx多层代理后，如何保留客户的真实ip呢

```
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header X-Real-IP $remote_addr;
```

第一行代码在层层代理时，保留住上级和本级代理的ip，如果只有一层代理，则保留客户端的ip和代理的ip，第二行可以保留客户端的ip。



