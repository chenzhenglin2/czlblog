# kafka常用命令

我这里zookeeper端口修改为12181 kafka端口为19092了

`cd  kafka/bin/`

- 列出所有topic

`./kafka-topics.sh   --list  --zookeeper localhost:12181 `

- 查看某个topic详情

`./kafka-topics.sh   --describe --zookeeper localhost:12181   --topic metric`

- 修改topic分区大小

`./kafka-topics.sh  --alter --zookeeper localhost:2181 --topic TransactionTopic --partitions 16`

- 查看所有groups

`./kafka-consumer-groups.sh --bootstrap-server localhost:19092  --list  `

- 查看groups详情，这个是动态的，一般可以作为是否采集到数据判断依据

`./kafka-consumer-groups.sh --bootstrap-server localhost:19092  --describe --group trx`