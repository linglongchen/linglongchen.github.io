
# RocketMQ总结





## 概念
### NameServer

### Topic
消息传输和存储的顶层容器，用于标识同一类业务逻辑的消息。
一个Topic的消息都会挂在一个CommitLog文件中。
![archifortopic-ef512066703a22865613ea9216c4c300.png](https://s2.loli.net/2024/08/28/YHs1jdO7tvqVroX.png)

### MessageQueue
消息队列是消息存储和传输的实际容器，也是 Apache RocketMQ 消息的最小存储单元。 Apache RocketMQ 的所有主题都是由多个队列组成，以此实现队列数量的水平拆分和队列内部的流式存储。

队列天然具备顺序性，即消息按照进入队列的顺序写入存储，同一队列间的消息天然存在顺序关系。

MessageQueue类似Kafka中的partition。

### Tag

### Message
消息是 Apache RocketMQ 中的最小数据传输单元。
生产者将业务数据的负载和拓展属性包装成Message发送到 Apache RocketMQ 服务端，服务端按照相关语义将消息投递到消费端进行消费。

消息大小不得超过其类型所对应的限制，否则消息会发送失败，系统默认的消息最大限制如下：
- 普通和顺序消息：4 MB
- 事务和定时或延时消息：64 KB

### Producer

### Consumer

### ConsumerGroup

### Subscription




## 流程


### 发送消息
![RocketMQ-producer.png](https://s2.loli.net/2024/08/28/iULKc9ODCjx1Xu7.png)

##### 整体流程
- 客户端启动时调用Producer#start启动Producer
- 客户端通过调用Producer发送消息，分为三种模式，异步发送、同步发送、单向发送
- 在Producer内部首先会查询topic信息、broker信息以及选择一个MessageQueue
### 存储消息


### 分发消息


### 拉取消息


### 消费消息






## 参考
https://rocketmq.apache.org/zh/docs/domainModel/04message

