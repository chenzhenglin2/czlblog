# NGINX指向静态网页

```yaml
  server {
      listen       8081;   #也可以指派其他端口
      server_name  localhost; 
      root /home/apptrace/tracing/dashboard;   #如果NGINX非root用户运行，不要放在root目录下
      autoindex on;             #开启索引功能
      autoindex_exact_size off; # 关闭计算文件确切大小（单位bytes），只显示大概大小（单位kb、mb、gb）
      autoindex_localtime on;   # 显示本机时间而非 GMT 时间
      location / {
        try_files $uri $uri/ /index.html;
     }
```

