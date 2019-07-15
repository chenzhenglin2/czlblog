#  Jvm 参数优化



## Zookeeper

修改方式：

`vi zookeeper/conf/java.env`

输入`export JVMFLAGS="$JVMFLAGS -Xms16g -Xmx16g"`

重启 Zookeeper 服务生效

## Kafka

修改方式：

`vi kafka/bin/kafka-server-start.sh`

将`export KAFKA_HEAP_OPTS="-Xmx1G -Xms1G"`中的 1G 修改为 32G

重启 Kafka 服务生效



- 这里具体调整多少，根据实际生产情况，如果是一个每天千万级别业务量，服务器内存大于64G，可以按照上面示例来调整。