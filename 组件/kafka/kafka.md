

## kafka环境搭建

 [1-1 消息队列和Kafka.pdf](..\..\..\零声Linux\kafka\1-Kafka应用与设计原理\1-1 消息队列和Kafka.pdf) 

 [1-2 Kafka使用场景与设计原理.pdf](..\..\..\零声Linux\kafka\1-Kafka应用与设计原理\1-2 Kafka使用场景与设计原理.pdf) 

 [1-3 kafka开发环境搭建.pdf](..\..\..\零声Linux\kafka\1-Kafka应用与设计原理\1-3 kafka开发环境搭建.pdf) 

 [1-4 C++操作kafka.pdf](..\..\..\零声Linux\kafka\1-Kafka应用与设计原理\1-4 C++操作kafka.pdf) 

## kafka实践

 [1-2 Kafka使用场景与设计原理v1.2.pdf](..\..\..\零声Linux\kafka\1-2 Kafka使用场景与设计原理v1.2.pdf) 

 [2-Kafka存储机制v1.1-预习.pdf](..\..\..\零声Linux\kafka\2-Kafka存储机制v1.1-预习.pdf) 

 [2-1-kafka C++实践.pdf](..\..\..\零声Linux\kafka\2-1-kafka C++实践.pdf) 

 [2-2-Kafka存储机制和可靠性.pdf](..\..\..\零声Linux\kafka\2-2-Kafka存储机制和可靠性.pdf) 

## 修改配置server.properties

```
broker.id=0//broker的id,-1自动分配
zookeeper.connect=192.168.124.134:2181//连接多个zookeeper需要在后面继续添加用逗号隔开
log文件路径可以改，默认路径在/tmp下，每次重启会清空
```

## list

```shell
sh kafka-topics.sh  --list --zookeeper localhost:2181
sh kafka-topics.sh 	--topic test --delete --zookeeper localhost:2181
sh kafka-topic.sh --topic test --describe --zookeeper localhost:2181
```

