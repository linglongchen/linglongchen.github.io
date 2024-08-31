
# RocketMQ总结





## 概念
### NameServer
为 Broker 提供服务发现和路由信息的组件。NameServer 维护一个路由表，客户端可以通过它来找到生产者和消费者的 Broker。
![Drawing 2024-08-31 23.02.36.excalidraw.png](https://s2.loli.net/2024/08/31/1EFuXsdqMkRx4IP.png)
### Broker
消息中间件的服务器节点，负责消息的存储和转发。一个集群中可以包含多个 Broker 实例。

### Topic
消息传输和存储的顶层容器，用于标识同一类业务逻辑的消息。
一个Topic的消息都会挂在一个CommitLog文件中。
![archifortopic-ef512066703a22865613ea9216c4c300.png](https://s2.loli.net/2024/08/28/YHs1jdO7tvqVroX.png)

### MessageQueue
消息队列是消息存储和传输的实际容器，也是 Apache RocketMQ 消息的最小存储单元。 Apache RocketMQ 的所有主题都是由多个队列组成，以此实现队列数量的水平拆分和队列内部的流式存储。

队列天然具备顺序性，即消息按照进入队列的顺序写入存储，同一队列间的消息天然存在顺序关系。

MessageQueue类似Kafka中的partition。

### Tag
标签是Apache RocketMQ 提供的细粒度消息分类属性，可以在主题层级之下做消息类型的细分。消费者通过订阅特定的标签来实现细粒度过滤。
![messagefilter0-ad2c8360f54b9a622238f8cffea12068.png](https://s2.loli.net/2024/08/31/eqvSnybLFi1DBhQ.png)
消费者获取消息时会触发服务端的动态过滤计算，Apache RocketMQ 服务端根据消费者上报的过滤条件的表达式进行匹配，并将符合条件的消息投递给消费者

### Message
消息是 Apache RocketMQ 中的最小数据传输单元。
生产者将业务数据的负载和拓展属性包装成Message发送到 Apache RocketMQ 服务端，服务端按照相关语义将消息投递到消费端进行消费。

消息大小不得超过其类型所对应的限制，否则消息会发送失败，系统默认的消息最大限制如下：
- 普通和顺序消息：4 MB
- 事务和定时或延时消息：64 KB

### Producer
生产者是 Apache RocketMQ 系统中用来构建并传输消息到服务端的运行实体。

### Consumer
消费者是 Apache RocketMQ 中用来接收并处理消息的运行实体。 消费者通常被集成在业务系统中，从 Apache RocketMQ 服务端获取消息，并将消息转化成业务可理解的信息，供业务逻辑处理。

### Subscription
订阅关系是 Apache RocketMQ 系统中消费者获取消息、处理消息的规则和状态配置。
通过配置订阅关系，可控制如下传输行为：
- 消息过滤规则：用于控制消费者在消费消息时，选择主题内的哪些消息进行消费，设置消费过滤规则可以高效地过滤消费者需要的消息集合，灵活根据不同的业务场景设置不同的消息接收范围
- 消费状态：Apache RocketMQ 服务端默认提供订阅关系持久化的能力，即消费者分组在服务端注册订阅关系后，当消费者离线并再次上线后，可以获取离线前的消费进度并继续消费


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

