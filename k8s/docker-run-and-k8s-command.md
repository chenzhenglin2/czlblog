## docker run  image 后面参数与kubernetes 的command对应关系

kubectl 与 Docker 命令关系 可以参考：http://docs.kubernetes.org.cn/70.html

docker run 与kubernetes （或docker-compose）配置对应关系

  像docker run image -args这种命令，args在kubernetes或docker-compose中怎么配置呢，如redis 镜像官网说明: ```docker  run  redis  --requirepass  passwd```

  运行一个带密码的容器；若在k8s的yaml、rancherUI（甚至是docker-compose）里面怎么配置呢，最简单办法 是查看这个镜像的dockerfile中的entrypoint ：docker-entrypoint.sh(只展示了部分代码)

```bash
if [ "${1#-}" != "$1" ] || [ "${1%.conf}" != "$1" ]; then 
 set -- redis-server "$@" 
 fi 
```



  从上面代码，可以看出如果参数不是“-”开头或者".conf"结尾，会自动变成redis-server + args（args为自定义的参数），若参数是“-”开头或“.conf”结尾，会直接用CMD命令+参数；所以在k8s（docker-compose)配置文件里面command的args里面可以直接跟参数，如果第一个参数写成了redis-server也没关系；

   如果你使用rancher，直接在 UI界面，点击编辑-高级就会显示命令(CMD)输入框，在右侧cmd里面填写
redis-server --requirepass 自己的密码，或 --requirepass 自己的密码  都可以，完成后可以验证，密码是否生效；
   如果没有找到镜像的dockerfile，当run镜像后，到容器中的默认的目录，查看是否有个可执行的二进制文件，然后在command 里面设置   “二进制文件  -args   ”但这个需要验证；

### 实战：

packetbeat 官网镜像 使用参考



```bash
docker run -d \

  --name=packetbeat \

  --user=packetbeat \

  --volume="/etc/localtime:/etc/localtime:ro" \

  --cap-add="NET_RAW" \

  --cap-add="NET_ADMIN" \

  --network=host \

  docker.elastic.co/beats/packetbeat:7.1.1 \

  --strict.perms=false -e \

  -E setup.kibana.host=ip:port \

  -E output.elasticsearch.hosts=ip:port
```



镜像后面的参数转换成配置文件

            args: [
              "--strict.perms=false",
              "-e",
              "-E","setup.kibana.host=ip:port",
              "-E","output.elasticsearch.hosts=ip:port"
            ]
如果使用rancher  直接UI上编辑 即可修改

![command](clipboard.png)