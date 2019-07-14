# mongodb安装

官网下载地址：https://repo.mongodb.org/yum/redhat/7/mongodb-org/3.6/x86_64/RPMS/  可以根据情况选择具体版本，下载五个安装包，分别是org mongos server shell tools 要保持每一个安装包版本一致。

mongodb安装非常简单，网上资料非常丰富，这里简单写了一个脚本来实现安装mongo，并修改端口、加密操作、启用密码验证等；



```
function install_mongo {
  sudo rpm  -qa | grep mongo &>/dev/null
  mongo_installed=$?
  if [[ ${mongo_installed} -eq 0 ]];then
    echo "mongod  already installed"
  else
    echo "Installing MongoDB 3.6.8..."
    set -e
    cd $DIR/pkg
    echo "Installing MongoDB RPM package..."
    sudo rpm -ivh  mongodb*.rpm
    echo "disable transparent hugepages..." #可要可不要
    sudo /bin/cp -rf disable-transparent-hugepages /etc/init.d/disable-transparent-hugepages
    sudo chmod 755 /etc/init.d/disable-transparent-hugepages
    sudo chkconfig --add disable-transparent-hugepages
    ###############################################
    echo "Tuning MongoDB system parameter..."

    echo "Config mongo to listen on all interfaces and config data dir"
    sudo chmod  a+rw  /etc/mongod.conf
    sudo sed -i   "s/bindIp: 127.0.0.1/bindIp: 0.0.0.0/g"  /etc/mongod.conf
    # 在REHL(redhat centos)系统中 mongodb 数据不能存储到根目录下
    root_dir="/root"
    if [[ $DIR =~ $root_dir*  ]]; then
      echo "directory in  root ,create new directory "
      mkdir -p /data/mongodb/{log,db}
      sudo chown  mongod:mongod -R /data/mongodb
      sudo chmod  777 -R /data
      sudo sed -i   "s|path: /var/log/mongodb/mongod.log|path: /data/mongodb/log/mongod.log|g"  /etc/mongod.conf
      sudo sed -i   "s|dbPath: /var/lib/mongo|dbPath: /data/mongodb/db|g"  /etc/mongod.conf
    else
      mkdir -p $MY_DIR/mongodb/{log,db}
      sudo chown  mongod:mongod -R $MY_DIR/mongodb
      sudo chmod  777 -R $MY_DIR/mongodb
      sudo sed -i   "s|path: /var/log/mongodb/mongod.log|path: $MY_DIR/mongodb/log/mongod.log|g"  /etc/mongod.conf
      sudo sed -i   "s|dbPath: /var/lib/mongo|dbPath: $MY_DIR/mongodb/db|g"  /etc/mongod.conf
    fi
    set +e
    echo "Config MongoDB to start on reboot..."
    sudo chmod -R 755 /usr/lib/systemd/system/mongod.service
    sudo systemctl enable mongod
    #配置mongo加密
    sudo setenforce 0
    sudo chmod a+x ~/
    echo "start mongodb..."
    sudo systemctl start mongod
    echo "config mongo security..."
    mongo admin --eval "db.setProfilingLevel(1, { slowms:\" 100000\" })"
    mongo admin --eval "db.createUser({user:\"admin\",customData:{description:\"superuser\"},pwd:\"MYPASSWD1\",roles:[{role:\"userAdminAnyDatabase\",db:\"admin\"}]})"
    sleep 3s
    mongo admin --eval "db.createUser({user:\"app\",pwd:\"MYPASSWD2\",roles:[\"root\"]})"
    echo "stop mongodb..."
    sudo systemctl stop mongod
    echo "Enable mongo security..."
    sudo sed -i '/#security/asecurity:\n  authorization: enabled' /etc/mongod.conf
    sudo sed -i   "s/port: 27017$/port: 17017/g"  /etc/mongod.conf
    sed  -ir '/Group=mongod/a\Restart=always'  /usr/lib/systemd/system/mongod.service
    mkdir -p $MY_DIR/config/mongod
    ln -s /etc/mongod.conf  $MY_DIR/config/mongod/mongod.conf
    echo "add port for mongo..."
    sudo firewall-cmd --add-port=17017/tcp --permanent
  fi
}
```

