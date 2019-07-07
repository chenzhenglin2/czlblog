# nginx实现负载均衡

```yaml
  upstream ygcr {
        server 192.16.35.30:38080 weight=1;
        server 192.16.33.20:38080 weight=1;
        ip_hash;  
        }
```



- 利用upstream（轮询） 可以进行负载均衡，通过weight值的大小决定权重。  ip_hash 来进行会话保持，保证同一个客户访问时 被调度到同一台服务器。

  # 反向代理

  ```
  server {
      listen       80;
      server_name  localhost;
      location ^~ /admin {
                  proxy_pass http://ygcr;
                  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                  proxy_set_header X-Real-IP $remote_addr;               
      }
  
      location ^~  /static/ {
                  proxy_pass http://ygcr;
                  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                  proxy_set_header X-Real-IP $remote_addr;
      }
      location = /product {
                  proxy_pass http://ygcr;
                  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                  proxy_set_header X-Real-IP $remote_addr;
      }
  }
  ```

  通过url请求访问此服务器，当端口后面是/admin、/static开头以及是/product的都转交给http://ygcr 来处理，ygcr这个在上面的负载均衡中已经定义。如访问http://ip:port/admin ，NGINX监听到这个端口后，匹配admin，然后交给http://ygcr 处理，最终访问的是 http://ygcr/admin 形成反向代理的效果。
