
# RocketMQ总结





## 概念
### NameServer
为 Broker 提供服务发现和路由信息的组件。NameServer 维护一个路由表，客户端可以通过它来找到生产者和消费者的 Broker。
![Drawing 2024-08-31 23.02.36.excalidraw.png](https://s2.loli.net/2024/08/31/1EFuXsdqMkRx4IP.png)
producer和consumer从NameServer获取broker的ip地址等信息，发送消息或者消费消息时直接从broker获取。因此broker同时和producer和consumer建立长连接。
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


### ConsumerQueue
关联消费者的队列数据，存在物理上的文件，文件中存储消息CommitLog的位点、消息长度、tag的hashCode。

![ConsumerQueue.png](https://s2.loli.net/2024/09/01/rmU9DC8VSqohsLK.png)
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

##### 核心逻辑：消费者发送消息给Broker

![RocketMQ-producer.png](https://s2.loli.net/2024/08/28/iULKc9ODCjx1Xu7.png)

##### 整体流程
- 客户端启动时调用Producer#start启动Producer
- 客户端通过调用Producer发送消息，分为三种模式，异步发送、同步发送、单向发送
- 在Producer内部首先会查询topic信息、broker信息以及选择一个MessageQueue
- 通过netty创建远程连接请求
- 通过netty调用远程请求，将数据发送到查询到的broker机器
- 处理发送响应结果


##### 源码梳理：
![-producerbroker.jpg](https://s2.loli.net/2024/08/31/ArhsQOpHi7eEDwX.jpg)
### 存储消息

##### 核心逻辑：Broker保存接收到Producer发送的消息

![Drawing 2024-09-01 09.50.59.excalidraw.png](https://s2.loli.net/2024/09/01/5a9zQhjCESkF7vw.png)
##### 整体流程：
1. Producer作为Netty客户端将消息通过channel发送到Broker
2. Broker作为Netty服务端监听指定端口接收Producer消息
3. Netty接收消息后会通过Message中的类型执行不同的Processor，接收消息的场景下使用SendMessageProcessor。
4. 消息会通过CommitLog文件进行存储，CommitLog存储相同topic下的所有的消息
5. 为了快速访问消息，避免文件过大，会将CommitLog拆分为不同的文件，即MappedFile
6. 消息最终会写入MappedFile保存在磁盘

##### 源码梳理：
![-brokerproducer.jpg](https://s2.loli.net/2024/08/31/AgSRvrLHaTopqtX.jpg)
### 分发消息

##### 核心逻辑：Broker将消息的位点分发到ConsumerQueue

![.png](https://s2.loli.net/2024/09/01/bW5JpkhoVEMPvid.png)
##### 整体流程：
1. Broker初始化分发消息的线程
2. 本地线程轮询CommitLg
3. 开始执行doDispatch分发逻辑
4. 通过topic+queueId确认唯一的ConsumerQueue
5. 获取ConsumerQueu的MappedFile
6. 通过MappedFile写入到磁盘

##### 源码梳理：
![-commitLogconsumerQueue.jpg](https://s2.loli.net/2024/08/31/5WjaPC6OtYJqXnx.jpg)
### 拉取消息
##### 核心逻辑：Consumer主动从Broker拉取消息
![.png](https://s2.loli.net/2024/09/02/3n7Bfq4RdV28NmK.png)
##### 整体流程：
1. Consumer内部初始化拉取线程
2. 线程通过Netty发起拉取消息的请求
3. Broker服务端接收请求处理
4. 拉取消息通过topic+queueId确认ConsumerQueue，拿到消息的位点offset
5. 通过offset在CommitLog中获取具体的消息体
6. 通过Netty响应给Consumer
7. Consumer接收到消息体后续执行消费

##### 源码梳理：
![-brokerconsumerconsumer.jpg](https://s2.loli.net/2024/09/02/Wpd4PoK7BecvtJH.jpg)
### 消费消息

##### 核心逻辑：Consumer消费消息
![.png](https://s2.loli.net/2024/09/02/GvlSHVkM5hwqB3C.png)
##### 核心流程：
1. Consumer主动拉取消息
2. 对拉取的消息做数量校验和大小校验
3. 选择消费模式，顺序模式ConsumeMessageOrderlyService和并发模式ConsumeMessageConcurrentlyService
4. 组装ConsumerRequest（本质是一个线程）
5. ConsumerReuqest执行业务消费逻辑，状态校验、更新位点


##### 源码梳理：
![-consumerbroker.jpg](https://s2.loli.net/2024/08/31/quJiGY7ActWSl9n.jpg)



## 总结
- 梳理RocketMQ中的核心概念
- 将RocketMQ的整体流程拆分为五个流程：
	- 发送消息流程
	- 保存消息流程
	- 分发消息流程
	- 拉取消息流程
	- 消费消息流程
- 每个流程梳理对应源码加深理解

## 参考
https://rocketmq.apache.org/zh/docs/domainModel/04message

