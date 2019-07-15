# Druid 安装与优化

以下是druid安装脚本，利用druid和tranquility压缩包进行安装，安装完成后，制作成系统服务。安装过程中有druid 初始化、jvm参数调整、与tranquility连接等操作。



```
function config_druid {
  if [[ -e /usr/lib/systemd/system/druid.service ]];then
    echo "druid already installed"
  else
    # 安装配置druid
    ###############################################
    cd $DIR
    mkdir -p ../{druid,tranquility}
    ###############################################
    echo "unzip druid package..."
    tar  -xzf  pkg/druid*.tar.gz  -C   ../druid   --strip-components 1
    tar  -xzf  pkg/tranquility*.tgz  -C   ../tranquility  --strip-components 1
    echo "config druid use right zookeeper..."
    sed -i 's/druid.zk.service.host=.*/druid.zk.service.host=localhost:12181/g' ../druid/conf-quickstart/druid/_common/common.runtime.properties
    echo "copy kafka.json..."
    /bin/cp -rf  ../kafka.json  ../tranquility
    # 这个json文件是配置与tranquility连接用的，根据生产情况来配置
    echo "add  start druid scripts"
    cat > ../druid/switch-druid <<EOF
#!/bin/bash -eu
usage="switch-druid (start|stop|status)"
if [ \$# -lt 1 ]; then
  echo \$usage
  exit 1
fi
action=\$1
dir=\$(dirname \$0)
cd \$dir
DRUID_CONF_DIR=conf-quickstart/druid ./bin/historical.sh \$action
DRUID_CONF_DIR=conf-quickstart/druid ./bin/broker.sh \$action
DRUID_CONF_DIR=conf-quickstart/druid ./bin/coordinator.sh \$action
DRUID_CONF_DIR=conf-quickstart/druid ./bin/overlord.sh \$action
DRUID_CONF_DIR=conf-quickstart/druid ./bin/middleManager.sh \$action
EOF

    cat > ../tranquility/start-tran <<EOF
#!/bin/bash -eu
dir=\$(dirname \$0)
nohup \$dir/../tranquility/bin/tranquility kafka -configFile \$dir/kafka.json \
-Djava.io.tmpdir=$dir/tmp -Ddruid.extensions.loadList='[]' >> \$dir/start-tran.log 2>&1 &
EOF

    echo "add chmod 777..."
    sudo chmod -R  777  ../druid
    sudo chmod -R  777  ../tranquility
    cd  ../druid/
    ./bin/init
    echo "configure druid broker jvm size"
    sed -i  "s#druid.processing.buffer.sizeBytes=.*#druid.processing.buffer.sizeBytes=104857600#g" \
    conf-quickstart/druid/broker/runtime.properties
    sed -i "s#MaxDirectMemorySize=.*#MaxDirectMemorySize=600m#g"  \
    conf-quickstart/druid/broker/jvm.config

    echo "configure druid historical jvm size"
    sed -i "s#druid.processing.buffer.sizeBytes=.*#druid.processing.buffer.sizeBytes=104857600#g"  \
    conf-quickstart/druid/historical/runtime.properties
    sed -i "s#-Xms.*#-Xms500m#g"    conf-quickstart/druid/historical/jvm.config
    sed -i "s#-Xmx.*#-Xmx700m#g"    conf-quickstart/druid/historical/jvm.config
    sed -i "s#MaxDirectMemorySize=.*#MaxDirectMemorySize=600m#g"    conf-quickstart/druid/historical/jvm.config

    echo "configure druid middleManager buffer size"
    sed -i '$a\druid.indexer.task.restoreTasksOnRestart=true' conf-quickstart/druid/middleManager/runtime.properties
    sed -i "s#druid.indexer.fork.property.druid.processing.buffer.sizeBytes=.*#druid.indexer.fork.property.druid.processing.buffer.sizeBytes=128000000#g" \
    conf-quickstart/druid/middleManager/runtime.properties
    sed -ir 's#druid.worker.capacity=.*#druid.worker.capacity=6#g' \
    conf-quickstart/druid/middleManager/runtime.properties

    echo "configure druid log level"
    sed -i "s#Root level=\"info\"#Root level=\"error\"#g"  conf-quickstart/druid/_common/log4j2.xml

    echo "set druid to service"
    cat > /usr/lib/systemd/system/druid.service <<EOF
[Unit]
Description=druid service
After=network.target remote-fs.target nss-lookup.target
After=zookeeper.service

[Service]
User=app
Group=app
Type=forking
ExecStart=$DIR/../druid/switch-druid start
#ExecReload=
ExecStop=$DIR/../druid/switch-druid stop
PrivateTmp=True
# file size
LimitFSIZE=infinity
# cpu time
LimitCPU=infinity
# virtual memory size
LimitAS=infinity
# open files
LimitNOFILE=64000
# processes/threads
LimitNPROC=64000
# locked memory
LimitMEMLOCK=infinity

[Install]
WantedBy=multi-user.target
EOF

    echo "set tranquility to service"
    cat > /usr/lib/systemd/system/tran.service <<EOF
[Unit]
Description=tranquility service
After=network.target remote-fs.target nss-lookup.target
After=zookeeper.service

[Service]
User=app
Group=apptrappe=forking
ExecStart=$DIR/../tranquility/start-tran
#ExecReload=
ExecStop=ps -ef | grep $APP_HOME/tranquility/bin | grep -v 'grep' | awk '{print \$2}'|xargs kill
PrivateTmp=True
# file size
LimitFSIZE=infinity
# cpu time
LimitCPU=infinity
# virtual memory size
LimitAS=infinity
# open files
LimitNOFILE=64000
# processes/threads
LimitNPROC=64000
# locked memory
LimitMEMLOCK=infinity

[Install]
WantedBy=multi-user.target
EOF

    sudo chmod   755  /usr/lib/systemd/system/druid.service
    sudo chmod   755  /usr/lib/systemd/system/tran.service
    sudo systemctl daemon-reload
    sudo systemctl enable druid.service
    sudo systemctl enable tran.service
    ln -s ../druid/conf-quickstart/druid/_common/  ../config/druid
    echo "add port for druid..."
    sudo firewall-cmd --add-port=8081/tcp --permanent
    sudo firewall-cmd --add-port=8082/tcp --permanent
    sudo firewall-cmd --add-port=8083/tcp --permanent
    sudo firewall-cmd --add-port=8084/tcp --permanent
    sudo firewall-cmd --add-port=8088/tcp --permanent
    sudo firewall-cmd --add-port=8090/tcp --permanent
    sudo firewall-cmd --add-port=8091/tcp --permanent
  fi
}

```

