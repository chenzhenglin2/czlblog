# 单台机器redis多实例

redis作为缓冲、验证码数据库非常好用，但redsi是单线程的，如果通过rpm工具安装的话，一台机器只能安装一个实例，如果有多个实例，需要部署多台机器上；下面脚本就实现了，一台机器安装5个redis，每个redis端口都不一样；端口从16379开始。



```bash
#!/bin/bash
# 告诉bash如果任何语句的执行结果不是true则应该退出
set -e

echo ""
echo "Installing redis_cluster...."
echo ""

# 获取系统环境变量
source ~/.bash_profile

# 获取脚本所在的目录名称，并进入该目录
DIR="$( cd "$( dirname "${BASH_SOURCE[0]}" )" && pwd )"
echo "$DIR"
cd $DIR/pkg
# 把redis制作成服务
echo "Installing redis..."

redis_version=`rpm  -qa|grep  redis`
if [[ ! -n "$redis_version" ]]; then
  if [[ ! -n $(uname -r | grep el7) ]]; then
    rpm -ivh jemalloc-3.6.0-1.el6.x86_64.rpm
    rpm -ivh  redis-4.0.1-2.el6.remi.x86_64.rpm
	else
	  rpm -ivh jemalloc-3.6.0-1.el7.x86_64.rpm
	  rpm -ivh redis-4.0.1-2.el7.remi.x86_64.rpm
	fi
	sudo sed -i "s/^bind *.*.*.*$/bind 0.0.0.0/g" /etc/redis.conf
	sudo sed -i "s/^save 900 1$/#save 900 1/g" /etc/redis.conf
	sudo sed -i "s/^save 300 10$/#save 300 10/g" /etc/redis.conf
	sudo sed -i "s/^save 60 10000$/#save 60 10000/g" /etc/redis.conf
	sudo sed -i "s/^#requirepass foobared$/requirepass RedisP@ssw0rd/g" /etc/redis.conf
	sudo sed -i 's/port 6379$/port 16379/g' /etc/redis.conf
fi
# 因为我们环境中 redis 存储不需落盘，直接把"save 900 1" 都注释掉了
echo  "复制redis配置文件,并且修改redis端口等配置"

sudo mkdir -p /etc/redis/
sudo mv /etc/redis.conf /etc/redis/16379.conf
sudo chmod 777 /etc/redis/16379.conf
sudo cp -r /etc/redis/16379.conf /etc/redis/16380.conf
sudo sed -i 's/^pidfile \/var\/run\/redis\/.*$/pidfile \/var\/run\/redis\/redis16380.pid/g' /etc/redis/16380.conf
sudo sed -i 's/^port 163.*$/port 16380/g' /etc/redis/16380.conf
sudo sed -i 's/^logfile \/var\/log\/redis\/redis.*$/logfile \/var\/log\/redis\/redis16380.log/g' /etc/redis/16380.conf
sudo sed -i 's/^dbfilename dump.*$/dbfilename dump16380.rdb/g' /etc/redis/16380.conf

sudo cp -r /etc/redis/16379.conf /etc/redis/16381.conf
sudo sed -i 's/^pidfile \/var\/run\/redis\/.*$/pidfile \/var\/run\/redis\/redis16381.pid/g' /etc/redis/16381.conf
sudo sed -i 's/^port 163.*$/port 16381/g' /etc/redis/16381.conf
sudo sed -i 's/^logfile \/var\/log\/redis\/redis.*$/logfile \/var\/log\/redis\/redis16381.log/g' /etc/redis/16381.conf
sudo sed -i 's/^dbfilename dump.*$/dbfilename dump16381.rdb/g' /etc/redis/16381.conf

sudo cp -r /etc/redis/16379.conf /etc/redis/16382.conf
sudo sed -i 's/^pidfile \/var\/run\/redis\/.*$/pidfile \/var\/run\/redis\/redis16382.pid/g' /etc/redis/16382.conf
sudo sed -i 's/^port 163.*$/port 16382/g' /etc/redis/16382.conf
sudo sed -i 's/^logfile \/var\/log\/redis\/redis.*$/logfile \/var\/log\/redis\/redis16382.log/g' /etc/redis/16382.conf
sudo sed -i 's/^dbfilename dump.*$/dbfilename dump16382.rdb/g' /etc/redis/16382.conf

sudo cp -r /etc/redis/16379.conf /etc/redis/16383.conf
sudo sed -i 's/^pidfile \/var\/run\/redis\/.*$/pidfile \/var\/run\/redis\/redis16383.pid/g' /etc/redis/16383.conf
sudo sed -i 's/^port 163.*$/port 16383/g' /etc/redis/16383.conf
sudo sed -i 's/^logfile \/var\/log\/redis\/redis.*$/logfile \/var\/log\/redis\/redis16383.log/g' /etc/redis/16383.conf
sudo sed -i 's/^dbfilename dump.*$/dbfilename dump16383.rdb/g' /etc/redis/16383.conf

if [[ ! -n $(uname -r | grep el7) ]]; then
  sudo chmod 777 /etc/rc.d/init.d/redis
  sudo sed -i 's/^REDIS_CONFIG="\/etc\/redis.*"$/REDIS_CONFIG="\/etc\/redis\/16379.conf"/g' /etc/rc.d/init.d/redis
  sudo chkconfig redis on
  sudo cp -r /etc/rc.d/init.d/redis /etc/rc.d/init.d/redis16380
  sudo sed -i 's/^pidfile="\/var\/run\/redis\/.*"$/pidfile="\/var\/run\/redis\/redis16380.pid"/g' /etc/rc.d/init.d/redis16380
  sudo sed -i 's/^REDIS_CONFIG="\/etc\/redis.*"$/REDIS_CONFIG="\/etc\/redis\/16380.conf"/g' /etc/rc.d/init.d/redis16380
  sudo chkconfig redis16380 on
  sudo cp -r /etc/rc.d/init.d/redis /etc/rc.d/init.d/redis16381
  sudo sed -i 's/^pidfile="\/var\/run\/redis\/.*"$/pidfile="\/var\/run\/redis\/redis16381.pid"/g' /etc/rc.d/init.d/redis16381
  sudo sed -i 's/^REDIS_CONFIG="\/etc\/redis.*"$/REDIS_CONFIG="\/etc\/redis\/16381.conf"/g' /etc/rc.d/init.d/redis16381
  sudo chkconfig redis16381 on
  sudo cp -r /etc/rc.d/init.d/redis /etc/rc.d/init.d/redis16382
  sudo sed -i 's/^pidfile="\/var\/run\/redis\/.*"$/pidfile="\/var\/run\/redis\/redis16382.pid"/g' /etc/rc.d/init.d/redis16382
  sudo sed -i 's/^REDIS_CONFIG="\/etc\/redis.*"$/REDIS_CONFIG="\/etc\/redis\/16382.conf"/g' /etc/rc.d/init.d/redis16382
	sudo chkconfig redis16382 on
	sudo cp -r /etc/rc.d/init.d/redis /etc/rc.d/init.d/redis16383
	sudo sed -i 's/^pidfile="\/var\/run\/redis\/.*"$/pidfile="\/var\/run\/redis\/redis16383.pid"/g' /etc/rc.d/init.d/redis16383
	sudo sed -i 's/^REDIS_CONFIG="\/etc\/redis.*"$/REDIS_CONFIG="\/etc\/redis\/16383.conf"/g' /etc/rc.d/init.d/redis16383
	sudo chkconfig redis16383 on

	sudo /sbin/iptables -I INPUT -p tcp --dport 16379 -j ACCEPT
	sudo /sbin/iptables -I INPUT -p tcp --dport 16380 -j ACCEPT
	sudo /sbin/iptables -I INPUT -p tcp --dport 16381 -j ACCEPT
	sudo /sbin/iptables -I INPUT -p tcp --dport 16382 -j ACCEPT
	sudo /sbin/iptables -I INPUT -p tcp --dport 16383 -j ACCEPT
	sudo /etc/rc.d/init.d/iptables save
	sudo service iptables restart

else
	sudo /bin/cp  -r redis\@.service   /usr/lib/systemd/system/
	cd  /usr/lib/systemd/system/
	sudo systemctl daemon-reload
	sudo systemctl enable redis@16379.service
	sudo systemctl enable redis@16380.service
	sudo systemctl enable redis@16381.service
	sudo systemctl enable redis@16382.service
	sudo systemctl enable redis@16383.service


  # 判断防火墙状态 如果结果为0表示防火墙为运行状态
  set +e
  sudo firewall-cmd --state 1>/dev/null 2>&1
  firewall_status=$?
  if [ $firewall_status -eq 0 ]; then
    sudo firewall-cmd --add-port=16379/tcp --permanent
    sudo firewall-cmd --add-port=16380/tcp --permanent
    sudo firewall-cmd --add-port=16381/tcp --permanent
    sudo firewall-cmd --add-port=16382/tcp --permanent
    sudo firewall-cmd --add-port=16383/tcp --permanent
    # 重启防火墙使其生效
    sudo firewall-cmd --reload
  fi
fi

echo  "启动多redis实例"
sleep 5s
sudo ./start-redis_cluster.sh

```

系统服务"redis@.service"   脚本，存放到/usr/lib/systemd/system/目录下 

```
[Unit]
Description=redis for %i
After=network.target

[Service]
ExecStart=/usr/bin/redis-server /etc/redis/%i.conf --daemonize no
ExecStop=/usr/bin/redis-shutdown /etc/redis/%i.conf
Restart=on-failure
RestartSec=1s
User=redis
Group=redis

[Install]
WantedBy=multi-user.target
```



增加一个启动脚本，因为单独一个个启动太麻烦。 对照着，也可以写一个停止脚本。

```
#!/bin/bash

if [[ ! -n $(uname -r | grep el7) ]]; then
	service redis  start
	service redis16380  start
	service redis16381  start
	service redis16382  start
	service redis16383  start
else
    sudo systemctl start  redis@16379
    sudo systemctl start  redis@16380
    sudo systemctl start  redis@16381
    sudo systemctl start  redis@16382
    sudo systemctl start  redis@16383
fi
```

