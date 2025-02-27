# Kafka

- [Kafka](#kafka)
  - [1、Kafka 基本概念](#1kafka-基本概念)
    - [1.1、什么是 Kafka](#11什么是-kafka)
      - [1.1.1、消息系统](#111消息系统)
      - [1.1.2、发布-订阅式与点对点式](#112发布-订阅式与点对点式)
    - [1.2、Kafka的组成](#12kafka的组成)
    - [1.3、Kafka的特点](#13kafka的特点)
  - [2、Kafka 原理](#2kafka-原理)
    - [2.1、Kafka 的管理和支持——Zookeeper](#21kafka-的管理和支持zookeeper)
    - [2.2、生产者 Producer](#22生产者-producer)
      - [2.2.1、数据生产流程](#221数据生产流程)
      - [2.2.2、创建生产者](#222创建生产者)
      - [2.2.3、生产者消息发送流程](#223生产者消息发送流程)
        - [main 线程](#main-线程)
          - [Producer 拦截器](#producer-拦截器)
          - [Serializer 序列化器](#serializer-序列化器)
          - [Partitioner 分区器](#partitioner-分区器)
          - [消息累加器 RecordAccumulator](#消息累加器-recordaccumulator)
        - [sender 线程](#sender-线程)
          - [发后即忘方式](#发后即忘方式)
          - [同步消息发送](#同步消息发送)
          - [异步消息发送](#异步消息发送)
          - [三种发送方式的特点对比](#三种发送方式的特点对比)
      - [2.2.4、重要生产者参数](#224重要生产者参数)
    - [2.3、节点 Broker](#23节点-broker)
      - [2.3.1、Broker 工作流程](#231broker-工作流程)
      - [2.3.2、Controller](#232controller)
        - [Controller 的选举过程](#controller-的选举过程)
        - [Controller的职责](#controller的职责)
      - [2.3.3、Topic](#233topic)
      - [2.3.4、Replica 副本](#234replica-副本)
        - [Leader 的选举](#leader-的选举)
        - [Leader 和 Follower 故障处理](#leader-和-follower-故障处理)
      - [2.3.5、文件存储](#235文件存储)
        - [Segment 文件命名规则](#segment-文件命名规则)
        - [Message 的结构](#message-的结构)
        - [文件清理策略](#文件清理策略)
      - [2.3.6、高效读写数据](#236高效读写数据)
      - [2.3.7、Broker 参数](#237broker-参数)
    - [2.4、消费者 Consumer](#24消费者-consumer)
      - [2.4.1、消费者组和分区重平衡](#241消费者组和分区重平衡)
        - [重平衡的优缺点](#重平衡的优缺点)
      - [2.4.2、 创建消费者](#242-创建消费者)
        - [主题订阅](#主题订阅)
        - [轮询消费](#轮询消费)
      - [2.4.3、消费者的配置](#243消费者的配置)
      - [2.4.5、提交和偏移量](#245提交和偏移量)
        - [自动提交](#自动提交)
        - [手动提交](#手动提交)
        - [指定 offset 消费](#指定-offset-消费)
        - [重复消费和漏消费](#重复消费和漏消费)
## 1、Kafka 基本概念

### 1.1、什么是 Kafka

> Kafka是一种分布式发布-订阅消息系统，主要用途就是实时数据流的处理和传输。

那么有个问题，什么是**消息系统**？

#### 1.1.1、消息系统

消息系统：又叫消息中间件，负责将数据从一个应用传递到另一个应用，应用只需关注数据，无需关注数据在两个或者多个应用之间是如何传递的。

常见的消息系统除了`Kafka`，常见的还有：
- RabbitMQ：一个基于AMQP协议的消息队列；
- ActiveMQ：开源的消息中间件，支持多种消息协议；
等等。

消息系统的特点：
1. 实时性：消息系统能够将消息实时的传递给用户或者系统，在金融或者电商领域，实时性尤为重要；
2. 异步性：消息的发送和消息处理分离，降低了各个组件之间的耦合性，消息发送者只需要关注消息的发送，消息的接收者只需要关注消息的处理。
3. 支持多种消息传递模式（点对点式和发布订阅式），同时支持多种消息处理格式；
4. 跨平台和多设备支持：可以实现跨平台的消息推送。

那么什么是**发布-订阅**呢？

#### 1.1.2、发布-订阅式与点对点式

消息系统的消息传递模式分为两种：一种是点对点（Point to Point,P2P）式，还有一种就是发布-订阅式，上面介绍的`RabbitMQ`和`ActiveMQ`就属于点对点式。

点对点式的特点就是：

1. 通过消息队列来实现，每条消息放入消息队列中，并且每条消息只能被一个消费者接收和处理。
2. 采用请求-响应模式，消息的发送和处理是一次性的。例如，在拼多多上购物时，订单处理系统中的支付请求。
3. 负载均衡：如果有多个消费者，消息会在消费者之间进行负载均衡，消费者的数量可以根据处理能力进行调整，但是被某个消费者消费过的消息就不能再被消费了。

点对点模式如下图：

![点对点模式](https://img2022.cnblogs.com/blog/2591061/202211/2591061-20221113153810167-372719945.png)


发布订阅模式如下图：

![发布订阅模式](https://img2022.cnblogs.com/blog/2591061/202211/2591061-20221113154118350-462490901.png)

点对点式和发布订阅式的区别
|特点|发布订阅式|点对点式|
|-|-|-|
|通信方式|通过消息代理进行间接通信|生产者直接发送给指定的消费者|
|耦合度|松耦合，各个组件或者模块的依赖关系较弱|紧耦合，各个组件或者模块的依赖关系较强|
|消息传递|异步|同步或者异步|
|消息消费|多订阅，一个消息可以被多个消费者消费|单订阅，一个消息只能被一个消费者消费|
|消息过滤|支持灵活的消息过滤，只接收和处理自己感兴趣的消息|不支持或者支持有限的消息过滤|
|可靠性和持久性|通常使用消息队列和主题来确保消息的可靠性和持久性|依赖于生产者和消费者的直接通信，可能缺乏可靠性保证|
|应用场景|适用于需要广播消息给多个消费者的场景。<li>租房信息订阅；<li>股票信息订阅；<li>在线购物网站的内容推荐|适用于需要确保消息按顺序被单个消费者处理的场景。<li>微信好友间聊天；<li>银行转账；|


### 1.2、Kafka的组成

Kafka 由以下部分组成：

|Kafka 组成|功能|特点|
|-|-|-|
|生产者`Producer`|将消息发送到指定的`topic`，并且可以指定消息发送到哪个`partition`|生产者发送消息是异步的，支持批量发送以提高性能。|
|消费者`Consumer`|通过订阅一个或多个`topic`，并从这些`topic`中拉取（pull）消息进行消费|消费者支持分组（`Group`）概念，一个消费者组内的消费者可以共同消费一个`topic`的多个`partition`，以实现负载均衡和容错，同时，一个分区只能被指定给一个消费。|
|消费者组`Consumer Group`|消费者组由一个或者多个消费者组成|<li> 同一个组中的消费者对于同一条消息只消费一次；<li>每个消费者都属于某个消费者组，如果不指定，那么所有的消费者都属于默认的组；<li>组内的所有消费者协调在一起来消费一个订阅主题( topic)的所有分区(partition)。当然，每个分区只能由同一个消费组内的一个消费者(consumer)来消费，可以由不同的消费组来消费；<li>|
|节点`Broker`|`Kafka`集群中的每个服务器节点被称为一个`Broker`，负责存储消息数据，并提供消息生产和消费的服务|`Broker`集群通过`Zookeeper`进行管理和协调，以实现高可用性和数据一致性|
|主题`Topic`|`Topic`是`Kafka`中用于存储消息的逻辑分类，每个`Topic`都包含多个`partition`，用于分散消息的存储和消费。主题类似于数据库的表。|<li>生产者向`Topic`发送消息，消费者从`Topic`订阅并消费消息；<li>不同主题的消息是物理隔离的；<li>`Topic`有一个或者多个分区；<li>同一个主题的消息保存在一个或者多个`Broker`上，用户只需要指定`Topic`即可以生产或者消费，不必关心存于何处。|
|分区`Partition`|`Partition`是`Topic`的物理分区，对应一个磁盘上的文件，用于实现消息的并行处理和存储|每个`Partition`可以有多个副本（`Replica`）以提高数据的可靠性和可用性|
|消息`message`|`Kafka`的数据单元称作消息，消息在文件中的位置称为`Offset`，`Offset`唯一标记一条`Message`|由字节数组成|
|批次`batch`|批次就是一组消息|同属于一个`Broker`和`Partition`|
|`Replica`副本|`Replica`是`Partition`的副本，用于实现数据的冗余和容错。每个`Partition`都有一个`Leader`副本和多个`Follower`副本。`Leader`副本负责处理读写请求(生产者发送数据的对象，消费者消费数据的对象)，`Follower`副本从`Leader`副本复制数据以保持数据一致性|当`Leader`副本出现故障时，`Kafka`会自动从`Follower`副本中选举出新的`Leader`副本，以保证服务的连续性|
|消息偏移量`offset`|表示分区`Partition`中每条消息的位置信息|是一个单调递增且不变的值|
|`Zookeeper`|`Zookeeper`不是`Kafka`的直接组成部分，但它在`Kafka`集群中扮演着至关重要的角色，管理`Kafka`集群的元数据（如`Broker`信息、`Topic`信息、`Partition`信息等），并协调`Broker`之间的交互和选举。|`Zookeeper`的高可用性和强一致性保证了`Kafka`集群的稳定性和可靠性|



### 1.3、Kafka的特点

`Kafka`具有以下特点：

1. 高吞吐量：`Kafka`的组成决定了其有高吞吐量的特点，可以处理非常高的消息吞吐量，适用于大规模数据处理和实时数据流；
2. 持久性：`Kafka`将消息持久化到磁盘中，即使消费者出现故障或者网络中断，消息也不会丢失；
3. 可靠性：`Kafka`通过副本机制保证消息的可靠性。每个`Partition`都有多个副本，分布在不同的`Broker`上，从而提高了数据的冗余性和容错性。即使某些节点发生故障，`Kafka`也能通过副本机制自动恢复数据的完整性;
4. 可伸缩性：`Kafka`采用分布式架构，可以方便地进行水平扩展，以应对不断增长的数据和负载需求。通过添加新的节点，可以线性扩展`Kafka`集群的处理能力，保证了系统的可靠性和性能;
5. 低延迟：`Kafka`设计用于实时数据流处理，因此具有较低的传输延迟；

`Kafka`的组成决定了其高吞吐量的特点，而高吞吐量又是其能够削峰的基础。`Kafka`作为中间件，可以将上游系统产生的突发流量暂存到`Kafka`的消息队列中，而不是直接发送到下游系统。这样，下游系统就可以按照自己的处理节奏，从`Kafka`中逐步消费这些消息，从而避免了因为突发流量而导致的系统崩溃。


同样地，削峰有助于实现高吞吐量：通过削峰，`Kafka`能够将突发的高峰流量平滑地分散到队列中，避免了因为流量洪峰而导致的系统崩溃或响应延迟。这样，系统就能够更加稳定地运行，并保持较高的吞吐量。


`Kafka`削峰的应用场景：
- 电商秒杀：由于用户会在短时间内集中访问系统，导致系统面临巨大的访问压力。通过引入`Kafka`作为消息队列，可以将用户的请求暂存到`Kafka`中，然后逐步推送给下游系统进行处理，从而避免了系统崩溃和响应延迟。
- 日志收集与处理：在大型系统中，日志的收集和处理是一个重要的任务。由于系统可能产生大量的日志数据，如果直接将这些数据推送到处理系统，可能会导致处理系统无法承受。通过Kafka进行削峰处理，可以将日志数据暂存到`Kafka`中，然后逐步推送给处理系统进行处理。


## 2、Kafka 原理

下面一张图是 Kafka 的基本架构，后面会从这张图出发，对 Kafka 进行深入了解。

![Kafka基础架构](https://img2022.cnblogs.com/blog/2591061/202211/2591061-20221113155143571-1630948393.png)


### 2.1、Kafka 的管理和支持——Zookeeper

`Zookeeper`是一个分布式协调服务。`Kafka`是使用的`Zookeeper`来构建的分布式服务。
其主要负责的功能包括：
1. 元数据管理：`Zookeeper`存储和管理`Kafka`集群的元数据，包括：`Topic`，`Partition`，`offset`以及`Replica`；
2. 集群管理：管理`Broker`的注册信息，当`Kafka`中有新的`Broker`加入或者离开时，`Zookeeper`负责通知和协调集群中的其他`Broker`，保持集群的一致性；
3. `Controller`选举：`Kafka`通过`Zookeeper`实现`Controller`选举，`Controller`将动态决策结果（如新 Leader、ISR 变更）写入`Zookeeper`，其他`Broker`通过监听`Zookeeper`节点变化（`Watcher`机制）同步最新元数据。


在`Kafka`中，`Zookeeper`以服务器的形式存在。但需要注意的是，这里的“服务器”并不是指它直接提供`Kafka`消息队列服务，而是指`Zookeeper`作为一个独立的进程或服务运行在服务器上，用于为`Kafka`集群提供分布式协调服务。

在实际部署中，`Zookeeper`服务器集群通常会与`Kafka`集群分开部署，以确保它们之间的独立性和可扩展性。

`Zookeeper`和`Kafka`的关系好比是，管理者和打工仔的关系，`Zookeeper`负责全局的协调和管理，`Kafka`负责处理具体的事务，相互协调使得分布式系统稳定运行。


### 2.2、生产者 Producer

正常的生产逻辑：
- 配置生产者客户端参数和创建相应的生产者实例
- 构建发送的消息
- 发送消息
- 关闭生产者实例

#### 2.2.1、数据生产流程


![数据生产流程](https://img2022.cnblogs.com/blog/2742789/202202/2742789-20220228223715656-1782209181.png)

对于一条记录，先对其进行序列化，然后根据 Topic 和 Partition，放进对应的发送队列中。如果 Partition 没填，分为两种情况：
- Key 有值，按照 Key 进行哈希，相同 Key 去一个Partition
- Key 无值，轮循选出 Partition

Producer 将会和 Topic 下所有 Partition Leader 保持 socket 连接，消息由 Producer 直接通过socket 发送到 Broker。其中 Partition Leader的位置注册在Zookeeper中，Producer作为Zookeeper Client，已经注册了 watch 用来监听 Partition Leader 的变更事件，因此，可以准确的知道谁是当前的 leader。

#### 2.2.2、创建生产者

要想 Kafka 中写入消息，首先需要创建一个生产者对象。Kafka 生产者有3个必选属性

1. bootstrap.servers: 该属性指定 Broker 的地址清单，清单中不需要包含所有的 Broker 地址，生产者会从给定的 Broker 地址中找到其他的 Broker 信息，不过建议至少提供 2 个 Broker 信息，一旦其中一个宕机，生产者仍然能够连接到集群上
2. key.serializer: 生产者需要知道采用何种方式把 Java 对象转换为字节数组。key.serializer 必须被设置为一个实现了 org.apache.kafka.common.serialization.Serializer 接口的类，生产者会使用这个类把键对象序列化为字节数组。Serializer 是一个接口，它表示类将会采用何种方式序列化，它的作用是把对象转换为字节，实现了 Serializer 接口的类主要有 ByteArraySerializer、StringSerializer、IntegerSerializer ，其中 ByteArraySerialize 是 Kafka 默认使用的序列化器。
3. value.serializer 指定的类会将值序列化

示例，创建 Kafka 生产者

        private Properties properties = new Properties();
        properties.put("bootstrap.servers","broker1:9092,broker2:9092");
        properties.put("key.serializer","org.apache.kafka.common.serialization.StringSerializer");
        properties.put("value.serializer","org.apache.kafka.common.serialization.StringSerializer");
        properties = new KafkaProducer<String,String>(properties);

#### 2.2.3、生产者消息发送流程

消息发送过程中涉及到 2 个线程。`main`线程和`Sender`线程。在`main`线程中创建了一个双端队列 RecordAccumulator。 `main`线程将消息发送给`RecordAccumulator`，`Sender`线程不断从 RecordAccumulator 中拉取消息发送到 Kafka Broker。

![数据发送流程](https://img2022.cnblogs.com/blog/2591061/202211/2591061-20221113182126109-2003618548.png)

##### main 线程


![main 线程](https://img2022.cnblogs.com/blog/2591061/202211/2591061-20221119152602831-1556480711.png)

![main 线程发送数据到缓冲区](https://img2022.cnblogs.com/blog/2591061/202211/2591061-20221119152918269-1015214468.png)

从创建一个`ProducerRecord`开始，这是 Kafka 的一个核心类。每写入一条数据，需要指定四个参数：Topic、Partition、Key 和 Value，其中 Topic 和 Value (要写入的数据)是必须要指定的，而 Key 和 Partition 是可选的。

###### Producer 拦截器

生产者拦截器可以直接在 producer.properties 文件中配置或者直接在代码中指定。

生产者拦截器的作用：
1. 对即将发送的数据进行修改或者增强，如：
   - 对消息内容进行加密或者解密
   - 为消息添加额外的元数据，如特定的头部信息
   - 修改消息的值、键或者消息的其他元数据
2. 消息的统计和监控：统计消息的发送情况、吞吐量、延迟等性能指标。
3. 性能优化：可以根据当前生产者的负载，动态调整发送频率和大小，以平衡吞吐量和延迟
4. 日志记录：可以记录每条消息的发送情况，便于排查问题。
5. 模拟故障：在一些测试环境中，拦截器可以模拟网络延迟或消息传递故障，以帮助测试系统的性能。
6. 消息过滤，某些特定情况下（如格式错误）拒绝发送某些消息到 Kafka

###### Serializer 序列化器 

经过生产者拦截器的`ProducerRecord`，由序列化器将这些键值对转化成为字节数组，这样才能够在网络上进行传输，数据到达分区器。

###### Partitioner 分区器

数据到达分区器之后，还需要确定向哪个分区发送消息。分区器通过 3 种策略决定消息要发送的分区。
1. `ProducerRecord`指定了分区号，直接使用该分区号；
2. `ProducerRecord`没有指定分区号，但是有 Key 值，使用 Key 的 Hash 函数映射指定一个分区，确保相同`Key`的消息分配到同一分区。
3. `ProducerRecord`既没有指定分区号，也没有 Key 值，将以轮询的方式选出一个分区，在较新版本的 Kafka 中，会采用粘性策略，直到当前批次达到阈值（batch.size 或 linger.ms）后，再切换到下一个分区。
4. 用户自定义分区策略，通过实现 `org.apache.kafka.clients.producer.Partitioner` 接口，可以定义自己的分区逻辑，显示配置生产者端的参数 Partitioner.class。

选择好分区之后，生产者就知道向哪个主题和分区发送消息了。


###### 消息累加器 RecordAccumulator

经过拦截器、序列化器、分区器之后的消息缓存到消息累加器`RecordAccumulator`中,`RecordAccumulator` 缓存的大小可以通过生产者客户端参数 buffer.memory 配置，默认值为 33554432B ，即32MB。 如果生产者发送消息的速度超过发送到服务器的速度 ，则会导致生产者空间不足，这个时候 KafkaProducer 的 `send（）`方法调用要么被阻塞，要么抛出异常，这个取决于参数 max.block.ms 的配置，此参数的默认值为 60000,即 60 秒 。

##### sender 线程

![sender 线程](https://img2022.cnblogs.com/blog/2591061/202211/2591061-20221119152805483-1834132623.png)

![sender 线程发送数据](https://img2022.cnblogs.com/blog/2591061/202211/2591061-20221119153038758-1123867300.png)


`sender`从`RecordAccumulator`中获取缓存的消息之后，会进一步将原本＜分区,Deque <Producer Batch＞＞ 的保存形式转变成 ＜Node , List< ProducerBatch＞的形式，其中 Node 表示 Kafka 集群的 broker 节点 

发送消息主要有以下几种方式：发后即忘方式、同步消息发送、异步消息发送

###### 发后即忘方式

生产者调用`send()`方法发送消息`ProducerRecord`对象，消息先被写入分区的缓冲区中，然后分批次发送给 Kafka Broker。

发送成功后，`send()`方法会返回一个`RecordMetadata`类型的`Future`对象。但是发后既忘方式并不处理这个`Future`对象，或者压根不返回这个对象。所以并没有办法知道消息发送是否成功。

###### 同步消息发送

同步发送的方式，使用`send()`方式之后，会立即调用`Future.get()`方法阻塞当前线程，等待服务器返回确认结果`RecordMetadata`，如果服务器返回错误，`get()` 方法会抛出异常，如果没有发生错误，我们会得到 RecordMetadata 对象，可以用它来查看消息记录。

###### 异步消息发送

同步发送消息都有个问题，那就是同一时间只能有一个消息在发送，这会造成许多消息无法直接发送，造成消息滞后，无法发挥效益最大化。

异步发送的方式，发送消息时注册回调函数`Callback`，消息发送后不阻塞，结果通过回调异步处理

###### 三种发送方式的特点对比

|发送方式|优点|缺点|
|-|-|-|
|发后即忘|吞吐量最高|可能丢失消息，仅适用于可靠性要求极低的方式，如可以容忍日志丢失类的场景|
|同步发送|吞吐量低|可靠性高，要求强一致性的场景|
|异步发送|吞吐量高|可靠性中等|


#### 2.2.4、重要生产者参数

上面大致介绍了 Kafka 生产者生产和发送消息的流程，下面分类介绍一些生产者的重要参数。

|必填参数|说明|示例|
|-|-|-|
|bootstrap.servers|生产者连接集群所需的broker地址清单。格式：`host1:port,host2:port`|`localhost:9092`|
|key.serializer|用于 key 键的序列化，它实现了 org.apache.kafka.common.serialization.Serializer 接口|StringSerializer|
|value.serializer|用于 value 值的序列化，实现了 org.apache.kafka.common.serialization.Serializer 接口|StringSerializer|

|核心性能优化参数|说明|示例|
|-|-|-|
|batch.size|单个批次（Batch）的大小（字节）。批次填满或达到 linger.ms 时间后发送。|默认值：16384 (16KB)，增大可提升吞吐量，但会增大延迟。|
|linger.ms|指定生产者发送 ProducerBatch 之前等待更多消息 (ProducerRecord) 加入 ProducerBatch 的时间|默认值：0（立即发送）增大可积累更多消息批量发送，提升吞吐量，但增加延迟。|
|buffer.memory|此参数用来设置生产者内存缓冲区的大小，生产者用它缓冲要发送到服务器的消息。如果应用程序发送消息的速度超过发送到服务器的速度，会导致生产者空间不足。|默认值：33554432 (32MB)|
|compression.type|指定消息压缩方式，默认 "none"，还可以配置为 "gzip", "snappy", "lz4"|默认值：none，启用压缩可减少网络传输量，提升吞吐，但增加 CPU 消耗。|
|max.in.flight.requests.per.connection|生产者在收到服务器响应之前可以发送多少消息，它的值越高，就会占用越多的内存，不过也会提高吞吐量。把它设为1 可以保证消息是按照发送的顺序写入服务器。|设为 1 可保证分区内消息顺序，但降低吞吐量；增大可提升并发，但可能导致服务端乱序（需结合 acks 配置）|
|max.request.size|限制生产者客户端能发送的消息的最大值。|默认 1M|

|可靠性参数|说明|示例|
|-|-|-|
|acks|指定分区中必须要有多少个副本接受这条消息，生产者才被认为写成功。|acks 参数有3种类型的值(都是字符串类型)：<li> acks="1" (默认)。生产者发送消息之后，只要分区的 leader 副本成功写入消息，那么它就会收到来自服务端的成功响应。 折中方案，可靠性和吞吐量都是中等。消息写入 leader 副本并 返回成功响应给生产者，且在被其他 follower 副本拉取之前 leader 副本崩溃，那么此时消息还是会丢失<li>acks="0" 。生产者发送消息之后不需要等待任何服务端的响应，可以达到最大吞吐量，但是可靠性不高。<li>acks="-1/all"。生产者在消息发送之后，需要等待 ISR 中的所有副本都成功 写入消息之后才能够收到来自服务端的成功响应。(最高可靠,leader 宕机也不丢失，生产者收到异常告知此次发送失败)，但是它的吞吐量最低|
|retries|发送失败后的重试次数。消息在从生产者发出到成功写入服务器之前可能发生一些临时性的异常，比如网络抖动、leader 副本的选举等，这种异常往往是可以自行恢复的，生产者可以通过配置 retries大于 0 的值，以此通过 内 部重试来恢复而不是一昧地将异常抛给生产者的应用程序，如果达到这个次数，生产者会放弃重试并返回错误|默认值：0，配置大于 0 的数|
|retry.backoff.ms|重试间隔时间（毫秒）。|默认值：100 ms，适当增大（如 500）避免频繁重试对 Broker 造成压力。|
|enable.idempotence|是否启用幂等性（保证消息仅发送一次）。|默认值：false，若需严格一次语义（Exactly-Once），需设为 true，并设置 acks=all|
|request.timeout.ms	|这个参数用来配置 Producer等待请求响应的最长时间|默认值为 30000 (ms)，网络延迟高时适当增大，但过大会降低故障感知速度|
|max.block.ms|此参数指定了在调用 `send()` 方法或使用 `partitionFor()` 方法获取元数据时生产者的最大阻塞时间或者当生产者的发送缓冲区已满的最大等待时间，生产者会抛出超时异常。|默认60000 ms（1min），若频繁超时，需检查 buffer.memory 或网络问题。|

|高级配置参数|说明|示例|
|-|-|-|
|partitioner.class|自定义分区器类（需实现 `org.apache.kafka.clients.producer.Partitioner`）|默认值：`DefaultPartitioner`，需按业务逻辑分配分区时使用（如根据业务 Key 哈希）。|
|connections.max.idle.ms|多久关闭闲置的连接|默认值：540000 ms|频繁创建连接影响性能时，可增大此值。|
|receive.buffer.bytes|指定了 TCP Socket 接收数据包的缓冲区的大小。|默认 32kb，设置为 -1，就使用操作系统的默认值|
|send.buffer.bytes|指定了 TCP Socket 发送数据包的缓冲区的大小|默认 128kb，设置为 -1，就使用操作系统的默认值|
|delivery.timeout.ms|消息从发送到收到 Broker 确认的总超时时间（包括重试）。|默认值 120000 ms，需大于 linger.ms + request.timeout.ms，避免未完成发送就超时。|


下面给出几个典型场景的生产者配置：

场景1：高吞吐量（日志收集）

    batch.size=65536       # 64KB
    linger.ms=50           # 等待50ms积累批次
    compression.type=lz4   # 高效压缩
    acks=1                 # 平衡可靠性
    buffer.memory=67108864 # 64MB
    max.in.flight.requests.per.connection=5 # 提升并发

场景2：高可靠性（金融交易）

    acks=all
    enable.idempotence=true
    retries=10
    retry.backoff.ms=500
    max.in.flight.requests.per.connection=1 # 保证顺序
    delivery.timeout.ms=180000              # 3分钟

场景3：低延迟（实时监控）

    linger.ms=0            # 立即发送
    batch.size=16384        # 小批次
    compression.type=none  # 避免压缩耗时
    max.in.flight.requests.per.connection=5
    request.timeout.ms=5000


### 2.3、节点 Broker

#### 2.3.1、Broker 工作流程

Broker 的工作流程如下：

![Broker 工作流程](https://img2022.cnblogs.com/blog/2591061/202211/2591061-20221114082355336-102953134.png)


1. Broker 启动后在 Zookeeper 中进行注册
2. Controller 谁先注册，谁说了算
3. 由选举出来的 Controller 监听 Broker 节点的变化
4. Controller 决定 Leader 选举
5. Controller 将节点信息上传到 Zookeeper 中
6. 其他 Controller 从 Zookeeper 中同步相关信息
7. 假设 Broker 1中的 Leader 挂掉了
8. Controller 监听到节点发生了变化
9. 获取 ISR
10. 选举新的 Leader （在isr中存活为前提，按照AR中排在前面的优先，例如：ar[1,0,2]，那么leader就会按照1,0,2的顺序轮询）
11. 更新 Leader 和 ISR

下面开始逐一介绍其中涉及到的知识点

#### 2.3.2、Controller

> Controller：是 Kafka 集群中的一个特殊的 Broker，它除了像普通 Broker那样对外提供消息的生产、消费、同步功能外，还额外承担了管理 Kafka 集群的 Broker、Topic、分区等职责。

Controller 负责管理这些元数据，那么它与 Zookeeper 的元数据管理有什么区别？

- Zookeeper：核心职责是可靠存储元数据和通知变更（通过 Watcher ）。它是被动的存储系统，不参与决策。
- Controller：作为集群的“大脑”，主动管理状态（如故障恢复、副本均衡）和执行决策（如 Leader 选举），并将结果持久化到 Zookeeper。

在后续新版的 Kafka 中，使用 KRaft 代替了 Zookeeper，实现了元数据管理的去中心化和性能优化。

##### Controller 的选举过程

`Kafka`集群中的每一个`Broker`都有可能成为`Controller`，具体的选举依赖于`Zookeeper`。

选举流程：

1. 候选者注册：在`Kafka`集群启动时，每个`Broker`都有可能成为`Controller`的候选者。各候选者会尝试在`Zookeeper`中创建临时有序临时节点（通常是`/controller`）来表明自己的参与。这些节点是临时的，意味着当`Broker`宕机或退出集群时，相应的节点会被自动删除。
  
2. 节点的排序与选举：`Zookeeper`对候选者结点进行排序，具有最小序号的节点成为新的`Controller`，如果多个候选者具有相同的最小序号，那么`Zookeeper`会根据节点的创建时间来选择最终的`Controller`，采用的是“先到先得”的抢占模式，`Zookeeper`会保证有且仅有一个`Broker`能创建成功，这个`Broker`就会成为集群的总控器`Controller`成功创建节点的 `Broker`将自己的`ID`、版本号等信息写入`/controller`节点。例如，节点内容可能为：`{"version":1, "brokerid":1001, "timestamp":"..."}`；

3. 选举完成：一旦选举完成，新的`Controller`节点将被选出，并且其他候选者`Broker`将知道哪个节点成为了新的`Controller`(监听机制)。新的`Controller`节点将负责管理`Kafka`集群的状态、执行分区分配、`Leader`选举等操作，会向`Zookeeper`中写入`epoch`值，即`Controller`的版本号，其余`Broker`在接收元数据变更时，会校验该版本号，拒绝旧的`Controller`的指令；

4. 故障转移和重新选举：当前的`Controller`节点发生故障或失效时，原先的`/controller`结点失效，`Kafka`集群会自动触发`Controller`的重新选举过程。这个过程由`Zookeeper`的临时节点和节点监听机制来保证。新的`Controller`候选者将尝试在`Zookeeper`中创建临时有序节点，参与新一轮的`Controller`选举过程。`ZooKeeper`将重新处理候选者节点，选举出新的`Controller`节点并更新`epoch`值，并接管集群管理任务，确保`Kafka`集群的正常运行和高可用性。


##### Controller的职责

1. 集群的状态管理：
   1. 为`Zookeeper`中的`/brokers/ids/`节点添加`BrokerChangeListener`，用来处理`Broker`增减的变化。监控集群中`Broker`的增减变化，包括处理`Broker`的加入，主动关闭和宕机，`Controller`需要及时更新集群元数据，并将集群变化通知到所有的`Broker`集群节点；
   2. 分区与副本管理：`Controller`负责管理和监控集群中所有分区的状态，包括`Topic`分区的创建、删除、状态转换以及副本的选举和状态更新。它通过`ZooKeeper`来协调这些任务，确保分区和副本的高可用性和一致性;

2. 元数据管理：`Controller`还负责维护集群的元数据信息，从`Zookeeper`中读取和更新集群的元数据信息，如`Topic`的分区信息、每个分区的`Leader`副本信息。当集群中的元数据发生变化时，`Controller`会及时更新集群元数据并将更新后的信息同步给集群中的所有`Broker`，确保每个`Broker`都能获取到最新的元数据信息；

3. 故障切换与内容复制：当分区的`Leader`副本发生故障时，`Controller`负责进行故障切换，选举新的`Leader`副本，并确保数据的正确复制和同步，具体选举过程可以参考下面的[Leader 的选举](#leader-的选举)；


`Kafka`分区和副本数据采用状态机方式进行管理，分区和副本的变化都在状态机内会引起状态机状态的变更，从而触发相应的变化事件：

- 分区状态机（管理 Topic 的分区，它有以下 4 种状态）：
  - NonExistentPartition：该状态表示分区没有被创建过或创建后被删除了
  - NewPartition：分区刚创建后，处于这个状态。此状态下分区已经分配了副本，但是还没有选举`leader`，也没有`ISR`列表
  - OnlinePartition：一旦这个分区的`leader`被选举出来，将处于这个状态
  - OfflinePartition：当分区的`leader`宕机，转移到这个状态
![分区状态机的切换](https://s6.51cto.com/oss/202104/09/f4d16e834f83f10f4f2cb4b860da35e9.png)

- 副本状态机（副本状态，管理分区副本信息，它也有 4 种状态）：
  - NewReplica: 创建`topic`和分区分配后创建`replicas`，此时，`replica`只能获取到成为`follower`状态变化请求。
  - OnlineReplica: 当`replica`成为`parition`的`assingned replicas`时，其状态变为 OnlineReplica, 即一个有效的OnlineReplica。
  - OfflineReplica: 当一个`replica`下线，进入此状态，这一般发生在`broker`宕机的情况下
  - NonExistentReplica: `Replica` 成功删除后，`replica` 进入 NonExistentReplica 状态

![副本状态机的切换](https://s2.51cto.com/oss/202104/09/de8f7c8bd4557cb535c4bf70d2afe0f6.png)

#### 2.3.3、Topic

Topic 有很多配置参数，有以下几种方式可以设置
- 创建 Topic 时的参数配置
- Kafka 的配置工具 kafka-topics.sh 中修改
- 通过 server.properties 中的默认配置

Topic 是 Kafka中用于存储消息的逻辑分类，每个 Topic 都包含多个 partition ，用于分散消息的存储和消费。主题类似于数据库的表。

#### 2.3.4、Replica 副本

Kafka 副本作用：提高数据可靠性。

Kafka 默认副本1个，生产环境一般配置为2个，保证数据可靠性；太多副本会增加磁盘存储空间，增加网络上数据传输，降低效率。

![](https://ask.qcloudimg.com/http-save/2039230/x8v603d95o.png)

> 副本 Replica：控制消息保存在几个 broker 上，一般情况下副本数小于等于 broker 的个数。

副本操作是以分区 Partition 为单位的。每个分区都有各自的 1 个主副本 Leader 和 N 个从副本 Follower；

以下是分区副本的概念
1. AR：分区中所有副本统称为AR(Assigned Replicas)；
2. ISR：所有与 Leader 副本保持一定程度同步的副本（包括 leader 副本）组成 ISR (In-Sync Replicas)，即当前可用的副本。
3. 与 Leader 副本同步之后过多的副本（不包括 Leader 副本）组成 OSR(Out-of-Sync Replicas)。 AR=ISR+OSR ，正常情况下 AR=ISR。

Follower 通过拉的方式从 Leader 同步数据。

注意：
- 消费者和生产者都是从 Leader 读写数据，不与 Follower 交互。
- 同一个副本不能放在同一个 Broker 中。

Kafka 中副本分为：Leader 和 Follower。Kafka 生产者只会把数据发往Leader，然后 Follower 找 Leader 进行同步数据。 

Kafka分区中的所有副本统称为AR（Assigned Repllicas），AR = ISR + OSR。

ISR，表示和 Leader 保持同步的 Follower 集合。如果 Follower 长时间未向 Leader 发送通信请求或同步数据，则该 Follower 将被踢出 ISR 。该时间阈值由 replica.lag.time.max.ms 参数设定，默认30s。Leader 发生故障之后，就会从ISR中选举新的Leader。

OSR，表示Follower与Leader副本同步时，延迟过多的副本

##### Leader 的选举

由于 Controller 监听了很多的 Zookeeper 结点，所以能够感知到 Broker 的存活。


> 副本优先机制：当分区 Leader 所在的 Broker 挂掉之后，Controller 会从每个 Partition 的 ISR 中取出第一个 Broker 里面的 Follower 作为 Leader。

![Leader选举流程](https://img2022.cnblogs.com/blog/2591061/202211/2591061-20221114082550997-1908598791.png)

##### Leader 和 Follower 故障处理

首先介绍两个术语

> LEO（Log End Offset）：每个副本的最后一个 Offset，LEO 就是最新的 Offset + 1

> HW（High Watermark）：所有副本中最小的 LEO。

Follower 故障处理机制：
1. Follower 被临时踢出 ISR
2. 这个期间 Leader 和 Follower 继续接收数据
![](https://img2022.cnblogs.com/blog/2591061/202211/2591061-20221114224019861-1484196173.png)
3. 待该 Follower 恢复后，Follower 会读取本地磁盘记录上次的 HW，并将 log 文件高于 HW 的部分截取掉，从 HW 开始向 Leader 进行同步。
![](https://img2022.cnblogs.com/blog/2591061/202211/2591061-20221114224233127-2143699298.png)
4. 等该 Follower 的 LEO 大于等于该 Partition 的 HW，即 Follower 追上 Leader 之后，就可以重新加入 ISR 了。
![](https://img2022.cnblogs.com/blog/2591061/202211/2591061-20221114224316112-1183425314.png)


Leader 故障处理机制：
1. Leader发生故障之后，会从ISR中选出一个新的Leader。
![](https://img2022.cnblogs.com/blog/2591061/202211/2591061-20221114224625266-595379941.png)
2. 为保证多个副本之间的数据一致性，其余的 Follower 会先将各自的 log 文件高于 HW 的部分截掉，然后从新的 Leader 同步数据。
![](https://img2022.cnblogs.com/blog/2591061/202211/2591061-20221114224746215-433624397.png)
注意：这只能保证副本之间的数据一致性，并不能保证数据不丢失或者不重复。

#### 2.3.5、文件存储



Topic 是逻辑上的概念，而 Partition 是物理上的概念，了解完基于 partition 的 Replica，再往深处了解一下 Partition。每个 partition 对应于一个 log 文件，该 log 文件中存储的就是Producer 生产的数据。Producer 生产的数据会被不断追加到该 log 文件末端。为防止 log 文件过大导致数据定位效率低下，Kafka采取了分片和索引机制，将每个 partition 分为多个 segment。每个 Segment 文件，包含三部分，一个是 .log 文件，一个是 .index 文件，还有一个是 .timeindex 文件，其中 .log 文件包含了发送的数据存储，.index 文件，记录的是 .log 文件的数据索引值，以便于加快数据的查询速度，.timeindex 文件是时间戳索引文件。

![](https://img2022.cnblogs.com/blog/2591061/202211/2591061-20221114230343330-264829160.png)

这些文件位于一个文件夹下，该文件夹的命名规则为：topic名称+分区序号，例如：first-0。

![.log 和 .index 文件](https://ask.qcloudimg.com/http-save/2039230/002wqqdvqv.png)

如上图，左半部分是索引文件 .index ，里面存储的是一对一对的 key-value ，其中 key 是消息在数据文件（对应的 .log 文件）中的编号，比如“1,3,6,8……”，分别表示在log文件中的第1条消息、第3条消息、第6条消息、第8条消息……

比如索引文件中 3,497 代表：数据文件中的第三个 Message，它的偏移地址为497。再来看数据文件中，Message 368772表示：在全局 Partiton 中是第368772个 Message。

index 文件中并没有为数据文件中的每条消息都建立索引，而是采用了稀疏存储的方式，每隔一定字节的数据建立一条索引。这样避免了索引文件占用过多的空间，从而可以将索引文件保留在内存中，通过内存映射（mmap）直接进行内存映射。但缺点是没有建立索引的 Message 也不能一次定位到其在数据文件的位置，从而需要做一次顺序扫描，但是这次顺序扫描的范围就很小了。

##### Segment 文件命名规则
Partition 全局的第一个 Segment 从0开始，后续每个 Segment 文件名为上一个全局 Partition 的最大 offset（偏移 Message 数）。数值最大为64位long大小，20位数字字符长度，没有数字就用 0 填充。

通过索引信息可以快速定位到 Message。通过 .index 元数据全部映射到内存，可以避免 Segment File的 IO 磁盘操作；通过索引文件稀疏存储，可以大幅降低 .index 文件元数据占用空间大小。

> 稀疏索引：为了数据创建索引，但范围并不是为每一条创建，而是为某一个区间创建；

- 优点：就是可以减少索引值的数量。
- 缺点：找到索引区间之后，要得进行第二次处理。

##### Message 的结构

生产者发送到 Kafka 的消息，都被 Kafka 包装成了 Message。

![Message的物理结构](https://ask.qcloudimg.com/http-save/2039230/5q1z5tmizh.png)

每条消息都被包装成上图这个结构，只有最后一个字段才是真正生产者发送的消息数据。


##### 文件清理策略

Kafka 中默认的日志保存时间为7天，可以通过调整如下参数修改保存时间。
- log.retention.hours，最低优先级小时，默认7天。
- log.retention.minutes，分钟。 
- log.retention.ms，最高优先级毫秒。 
- log.retention.check.interval.ms，负责设置检查周期，默认5分钟。

Kafka 中提供的日志清理策略有 delete 和 compact 两种。 
- delete日志删除：将过期数据删除。log.cleanup.policy = delete。所有数据启用删除策略。
  - 基于时间：默认打开。以segment中所有记录中的最大时间戳作为该文件时间戳。
  - 基于大小：默认关闭。超过设置的所有日志总大小，删除最早的 segment。log.retention.bytes，默认等于-1，表示无穷大。 
- compact 日志压缩：对于相同 key 的不同 value 值，只保留最后一个版本。log.cleanup.policy=compact 所有数据启用压缩策略。

![](https://img2022.cnblogs.com/blog/2591061/202211/2591061-20221114234826831-894934108.png)

压缩后的 offset 可能是不连续的，比如上图中没有 6，当从这些 offset 消费消息时，将会拿到比这个 offset 大的 offset 对应的消息，实际上会拿到 offset 为7的消息，并从这个位置开始消费。

这种策略只适合特殊场景，比如消息的key是用户ID，value是用户的资料，通过这种压缩策略，整个消息集里就保存了所有用户最新的资料。


#### 2.3.6、高效读写数据

1. Kafka是分布式集群，可以采用分区技术，并行度高。
2. 读数据采用稀疏索引，可以快速定位到要消费的数据。
3. 顺序写磁盘。
4. 页缓存 + 零拷贝技术

> 零拷贝：Kafka 的数据加工处理操作交由 Kafka 生产者和 Kafka 消费者处理。Kafka Broker 应用层不关心存储的数据，所以就不用走应用层，传输效率高。

> PageCache 页缓存：Kafka 重度依赖底层操作系统提供的 PageCache 功能。当上层有写操作时，操作系统只是将数据写入PageCache。当读操作发生时，先从 PageCache 中查找，如果找不到，再去磁盘中读取。实际上 PageCache 是把尽可能多的空闲内存都当做了磁盘缓存来使用。

![](https://img2022.cnblogs.com/blog/2591061/202211/2591061-20221115212311764-1676882155.png)

涉及到的两个参数为：
- log.flush.interval.messages 	强制页缓存刷写到磁盘的条数，默认是long的最大值，9223372036854775807。一般不建议修改，交给系统自己管理。
- log.flush.interval.ms	每隔多久，刷数据到磁盘，默认是null。一般不建议修改，交给系统自己管理。

#### 2.3.7、Broker 参数

Broker 有一些重要的参数，主要是在 server.properties 配置文件中设置的。这个文件是每个 Kafka Broker 节点的核心配置文件，通常位于 Kafka 安装目录下的 config 目录中，下面详细说明

|基础参数|说明|示例|
|-|-|-|
|broker.id|每个 Kafka Broker 的唯一标识符|整数，通常按照顺序分配|
|listeners|指定 Kafka Broker 启动时监听的协议和端口。|格式为 `protocol://host:port`。如果要配置多个监听器，可以使用逗号分隔，如果使用配置样本启动 Kafka，会监听 9092 端口，例如 `listeners=PLAINTEXT://localhost:9092`|
|zookeeper connect|用于保存 Broker 元数据的 Zookeeper 地址是通过 zookeeper.connect 来指定的。|如指定 `localhost:2181` 则表示这个 Zookeeper 是运行在本地 2181 端口上的。如通过 `zk1:2181,zk2:2181,zk3:2181` 来指定 zookeeper.connect 的多个参数值。该配置参数是用冒号分割的一组 `hostname:port/path` 列表，其含义如下:<li>`hostname` 是 Zookeeper 服务器的机器名或者 ip 地址；<li>`port` 是 Zookeeper 客户端的端口号;<li>`/path` 是可选择的 Zookeeper 路径，如果不指定默认使用根路径|


|性能优化参数|说明|示例|
|-|-|-|
|num.network.threads|处理网络请求的线程数（接收客户端请求）。|根据 CPU 核心数和网络负载调整（通常设为 CPU 核心数的 1.5-2 倍）|
|num.io.threads|负责写磁盘的线程数。整个参数值要占总核数的50%。|默认值：8|
|num.replica.fetchers|副本从 Leader 同步数据的线程数。|这个参数占总核数的50%的1/3 。|
|log.flush.interval.messages|累积多少条消息后刷新到磁盘|可靠性要求高时设为较小值（如 10000），但会降低吞吐量。|
|log.flush.interval.ms|每隔多久，刷数据到磁盘。|默认是null。一般不建议修改，交给系统自己管理。|
|socket.send.buffer.bytes / socket.receive.buffer.bytes|TCP 发送/接收缓冲区大小（字节）。|网络带宽高时增大（如 1MB）|

|存储管理参数|说明|示例|
|-|-|-|
|log.dirs|Kafka 把所有的消息都保存到磁盘上，存放这些日志片段的目录是通过 log.dirs 来制定的，它是用一组逗号来分割的本地系统路径。|log.dirs 是没有默认值的，必须手动指定。比如通过 `/home/kafka1,/home/kafka2,/home/kafka3` 这样来配置这个参数的值|
|log.retention.hours|日志保留时间（小时）|默认：168 (7天)	根据存储容量和业务需求调整（如 72 小时）。|
|log.retention.bytes|单个分区的最大日志大小（字节），超过则删除旧数据。|默认等于-1，表示无穷大。结合 log.retention.hours 使用|
|log.segment.bytes|单个日志段文件的大小（字节）。|默认值：1073741824 (1GB)	增大可减少段文件数量，但延长索引重建时间（建议 1-10GB）。|
|log.index.interval.bytes|kafka里面每当写入了指定大小的日志（.log），然后就往 index 文件里面记录一个索引。稀疏索引。|默认4kb|
|log.cleanup.policy |日志清理策略。|默认值：delete。<li>delete：基于时间或大小删除。<li>compact：压缩保留最新键值。<br>需要保留键最新值时设为 compact（如事务日志）。|
|num.recovery.threads.per.data.dir|以下 3 种情况，Kafka 会使用可配置的线程池来处理日志片段<li>服务器正常启动，用于打开每个分区的日志片段；<li>服务器崩溃后重启，用于检查和截断每个分区的日志片段；<li>服务器正常关闭，用于关闭日志片段。<br>默认情况下，每个日志目录仅使用一个线程。由于这些线程仅在服务器启动和崩溃时使用，所以可以设置大量的线程达到并行操作的目的。这样一旦某个服务器发生崩溃，可以使用并行操作节约时间。|注意配置时的数字对应的是 log.dirs 指定的单个日志目录，即如果 log.dirs 指定了3个路径，num.recovery.threads.per.data.dir 被设为 8，则总共需要 24 个线程。|


|高可用和副本参数|说明|示例|
|-|-|-|
|default.replication.factor|默认副本数（创建主题时未指定则使用此值）。|默认值：1，生产环境建议 3，确保冗余|
|min.insync.replicas|最小同步副本数（ISR），影响生产者 acks=all 时的可用性。|默认值：1|
|unclean.leader.election.enable|是否允许非同步副本（非 ISR）成为 Leader。|默认 false，保持 false，避免数据不一致。|
|controller.socket.timeout.ms|控制器（Controller）与 Broker 通信的超时时间（毫秒）。|默认值：30000 (30秒)，网络不稳定时适当增大|
|offsets.topic.replication.factor|__consumer_offsets 主题的副本数。|默认值：3，保持和default.replication.factor 一致。|
|auto.create.topics.enable|是否允许自动创建 Topic|默认情况下，Kafka 会采用3种方式创建 Topic：<li>当一个生产者开始往主题写入消息时<li>当一个消费者开始从主题读取消息时<li>当任意一个客户端向主题发送元数据请求时<br>auto.create.topics.enable 参数建议设置成 false，即不允许自动创建 Topic|











### 2.4、消费者 Consumer

一个正常的消费逻辑需要具备以下几个步骤：

1. 配置消费者客户端参数及创建相应的消费者实例。
2. 订阅主题。
3. 拉取消息并消费。
4. 提交消费位移。
5. 关闭消费者实例。


消息经由生产者发送到 Kafka Broker 的 Topic 之后，就会被消费者订阅消费。应用程序首先需要创建一个 KafkaConsumer 对象，订阅主题并开始接受消息，验证消息并保存结果。

一段时间后，生产者往主题写入的速度超过了应用程序验证数据的速度，如果只使用单个消费者的话，应用程序会跟不上消息生成的速度，像多个生产者向相同的主题写入消息一样，这时候就需要多个消费者共同参与消费主题中的消息，就是消费者组的概念。


有两种消费方式：
- 一个消费者群组消费一个主题中的消息，这种消费模式又称为点对点的消费方式，点对点的消费方式又被称为消息队列
- 一个主题中的消息被多个消费者群组共同消费，这种消费模式又称为发布-订阅模式


#### 2.4.1、消费者组和分区重平衡


一个群组中的消费者订阅的都是相同的主题，每个消费者接收主题一部分分区的消息。
![](https://img2022.cnblogs.com/blog/2591061/202211/2591061-20221115213900917-1827632670.png)
![](https://img2022.cnblogs.com/blog/2591061/202211/2591061-20221115214037175-867628172.png)


向群组中增加消费者是横向伸缩消费能力的主要方式。可以通过增加消费组的消费者来进行水平扩展提升消费能力。

这也是为什么建议创建主题时使用比较多的分区数，这样可以在消费负载高的情况下增加消费者来提升性能。另外，消费者的数量不应该比分区数多，因为多出来的消费者是空闲的，没有任何帮助。

Kafka 还有个特性，消息只写入一次，每个应用都可以消费到所有的信息。

通过上面图片的分析，可以得出

最初是一个消费者订阅一个主题并消费其全部分区的消息，后来有一个消费者加入群组，随后又有更多的消费者加入群组，而新加入的消费者实例分摊了最初消费者的部分消息，这种把分区的所有权通过一个消费者转到其他消费者的行为称为**重平衡（Rebalance）** 。

![](https://camo.githubusercontent.com/3a3db52c1799431b373b27b21ee9ac4cf8811fd808ce1fe96f3ac7c43aec79f4/68747470733a2f2f696d67323031382e636e626c6f67732e636f6d2f626c6f672f313531353131312f3230313931312f313531353131312d32303139313132383132343630333237372d3433313032353935312e706e67)


##### 重平衡的优缺点

|重平衡|说明|
|-|:-|
|优点|为消费者组带来了高可用性和伸缩性，可以放心的添加或者删除消费者|
|缺点|<li>在重平衡期间，消费者无法读取消息，造成整个消费者组在重平衡的期间都不可用。<li>当分区被重新分配给另一个消费者时，消息当前的读取状态会丢失，它有可能还需要去刷新缓存，在它重新恢复状态之前会拖慢应用程序|

消费者通过向 组织协调者（Broker） 发送心跳来维护自己是消费者组的一员并确认拥有的分区。对于不同不的消费群体来说，其组织协调者可以是不同的。只要消费者定期发送心跳，就会认为消费者是存活的并处理其分区中的消息。当消费者检索记录或者提交它所消费的记录时就会发送心跳。

如果过了一段时间消费者停止发送心跳了，会话（Session）就会过期，组织协调者就会认为这个 Consumer 已经死亡，就会触发一次重平衡。如果消费者宕机并且停止发送消息，组织协调者会等待几秒钟，确认它死亡了才会触发重平衡。

在这段时间里，死亡的消费者将不处理任何消息。在清理消费者时，消费者将通知协调者它要离开群组，组织协调者会触发一次重平衡，尽量降低处理停顿。

**也就是说，在重平衡期间，消费者组中的消费者实例都会停止消费，等待重平衡的完成。而且重平衡这个过程很慢......**


所以为了避免频繁的重平衡，可以得出下面的关系

> Partition 数量决定了每个 Consumer Group 中并发消费者的最大数量。

1. 某一个主题下的分区数，对于消费该主题的同一个消费组下的消费者数量，应该小于等于该主题下的分区数。如：某一个主题有4个分区，那么消费组中的消费者应该小于等于4，而且最好与分区数成整数倍 1、2、4 。
2. 同一个分区下的数据，在同一时刻，不能同一个消费组的不同消费者消费。
3. 分区数越多，同一时间可以有越多的消费者来进行消费，消费数据的速度就会越快，提高消费的性能。

#### 2.4.2、 创建消费者

![](https://img2022.cnblogs.com/blog/2591061/202211/2591061-20221119153827376-699170512.png)

在读取消息之前，需要创建一个 KafkaConsumer 对象，这个对象和 KafkaProducer 对象十分像。使用3个属性就足矣，分别是 bootstrap.server，key.deserializer，value.deserializer。还有一个属性是 group.id 这个属性不是必须的，它指定了 KafkaConsumer 是属于哪个消费者群组。

    Properties properties = new Properties();
    properties.put("bootstrap.server","192.168.1.9:9092");     
    properties.put("key.serializer","org.apache.kafka.common.serialization.StringSerializer");  
    properties.put("value.serializer","org.apache.kafka.common.serialization.StringSerializer");
    KafkaConsumer<String,String> consumer = new KafkaConsumer<>(properties);

##### 主题订阅

![](https://img2022.cnblogs.com/blog/2591061/202211/2591061-20221119153937354-2097062738.png)

创建好消费者之后，就使用 `subscribe()` 方法接受一个主题列表作为参数，使用起来比较简单

    consumer.subscribe(Collections.singletonList("customerTopic"));

参数传入的是一个正则表达式，只订阅了一个 customerTopic，参数出入一个正则表达式，如果创建了新的主题，并且主题的名字和正则表达式相匹配，那么会立即触发一次重平衡，消费者就可以读取新的主题。

##### 轮询消费

![](https://img2022.cnblogs.com/blog/2591061/202211/2591061-20221119154033722-394421511.png)

Kafka 生产者产生的消息消费者是不知道的。Consumer 采用的是轮询的方式定期去 Kafka Broker 中进行数据的检索，如果有数据就用来消费，没有数据就继续轮询等待。

    try {
    while (true) {
      ConsumerRecords<String, String> records = consumer.poll(Duration.ofSeconds(100));
      for (ConsumerRecord<String, String> record : records) {
        int updateCount = 1;
        if (map.containsKey(record.value())) {
          updateCount = (int) map.get(record.value() + 1);
        }
        map.put(record.value(), updateCount);
      }
    }
  }finally {
    consumer.close();
  }

第三行代码表示，Kafka 必须定期循环请求数据，否则就会认为该 Consumer 已经挂了，会触发重平衡，它的分区会移交给群组中的其它消费者。传给`poll()`方法的是一个超时时间，用 java.time.Duration 类来表示，如果该参数被设置为 0 ，`poll()` 方法会立刻返回，否则就会在指定的毫秒数内一直等待 Broker 返回数据。
`poll()` 方法会返回一个记录列表。每条记录都包含了记录所属主题的信息，记录所在分区的信息、记录在分区中的偏移量，以及记录的键值对。一般会遍历这个列表，逐条处理每条记录。

在退出应用程序之前使用`close()`方法关闭消费者。网络连接和 socket 也会随之关闭，并立即触发一次重平衡，而不是等待群组协调器发现它不再发送心跳并认定它已经死亡。


#### 2.4.3、消费者的配置

还是和生产者一样，对这些参数进行分类

|必填参数|说明|示例|
|-|-|-|
|bootstrap.servers|消费者连接集群所需的broker地址清单。格式：`host1:port,host2:port`|`localhost:9092`|
|key.serializer|用于 key 键的反序列化，它实现了 org.apache.kafka.common.serialization.Deserializer 接口|StringSerializer|
|value.serializer|用于 value 值的反序列化，实现了 org.apache.kafka.common.serialization.Deserializer 接口|StringSerializer|
|group.id|消费者组 ID，同一组内的消费者共同消费同一主题的分区|my-consumer-group|


|核心消费控制参数|说明|示例|
|-|-|-|
|auto.offset.reset|指定了消费者在读取一个没有偏移量的分区或者偏移量无效的情况下的该如何处理。|它的默认值是 latest<li>- earliest：从最早偏移量开始消费。<li>- latest：从最新偏移量开始消费。<li>- none：抛出异常。|
|enable.auto.commit|指定了消费者是否自动提交偏移量|默认值是 true。为了尽量避免出现重复数据和数据丢失，可以把它设置为 false，由自己控制何时提交偏移量。如果把它设置为 true，还可以通过 auto.commit.interval.ms 属性来控制提交的频率|
|auto.commit.interval.ms|自动提交偏移量的间隔时间（毫秒）。|默认值：5000 (5秒)。缩短间隔可减少重复消费风险，但增加 Broker 负载；增大间隔提高吞吐量。|
|max.poll.records|单次 poll() 调用返回的最大消息数。|默认值：500，设置时避免单次处理过多消息导致超时。|
|max.poll.interval.ms|两次 poll() 调用的最大间隔时间（毫秒），超时则消费者被踢出组触发重平衡。|默认值：300000 (5分钟)。若消息处理耗时较长（如复杂计算），需增大此值以避免误判消费者失效。|
|session.timeout.ms|指定了消费者可以多久不发送心跳。消费者与 Broker 会话超时时间（毫秒），超时则消费者被踢出组，就会触发重平衡。|默认值：3000 (3秒)。需大于 heartbeat.interval.ms 的 3 倍。设置的比默认值小，可以更快地检测和恢复崩愤的节点，不过长时间的轮询或垃圾收集可能导致非预期的重平衡。把该属性的值设置得大一些，可以减少意外的重平衡，不过检测节点崩溃需要更长的时间。|
|heartbeat.interval.ms|消费者发送心跳给 Broker 的间隔时间（毫秒），一般这个参数和 session.timeout.ms 一起改。|默认值：1000 (1秒)。缩短间隔可更快检测失效消费者，但增加网络开销；通常保持默认。|


|性能优化参数|说明|示例|
|-|-|-|
|fetch.min.bytes|指定了消费者从服务器获取记录的最小字节数。Broker 在收到消费者的数据请求时，如果可用的数据量小于 fetch.min.bytes 指定的大小，那么它会等到有足够的可用数据时才把它返回给消费者。这样可以降低消费者和 Broker 的工作负载|默认值：1KB。增大可减少请求次数，提高吞吐量，但增加延迟。|
|fetch.max.bytes|消费者单次 fetch 请求从服务器获取记录的最大数据量（字节）。|默认值：50MB。 应避免单次请求过大|
|fetch.max.wait.ms|指定 Broker 的等待时间，当一直无法满足 fetch.min.bytes 时，就会导致延时，当 Kafka 收到消费者的请求时，就会判断 fetch.min.bytes 和 fetch.max.wait.ms 哪个先满足|默认是 500 ms，为了避免潜在的延时，可以将该值设置的小一点|
|max.partition.fetch.bytes|指定了服务器从每个分区里返回给消费者的最大字节数。<li>`KafkaConsumer.poll()`方法从每个分区里返回的记录最多不超过 max.partition.fetch.bytes 指定的字节<li>max.partition.fetch.bytes 的值必须比 broker 能够接收的最大消息的字节数(通过 max.message.size 属性配置大)，否则消费者可能无法读取这些消息，导致消费者一直挂起重试。<li>如果消费者单次调用`poll()`返回的数据太多，消费者需要更多的时间进行处理，可能无法及时进行下一个轮询来避免会话过期。如果出现这种情况，可以把 max.partition.fetch.bytes 值改小，或者延长会话过期时间|默认值：1MB|
|connections.max.idle.ms|空闲连接关闭时间（毫秒）。|默认值：540000 (9分钟)。频繁重连影响性能时可适当增大。|

|分区分配策略|说明|示例|
|-|-|-|
|partition.assignment.strategy|分区分配给消费者的策略类（可配置多个，逗号分隔）：<li>RangeAssignor：按范围分配。<li>RoundRobinAssignor：轮询分配。<li>StickyAssignor：粘性分配（减少重平衡时的分区迁移）。|默认值：RangeAssignor。StickyAssignor 适合消费者数量频繁变化的场景，减少重平衡开销。|


|高级配置|说明|示例|
|-|-|-|
|client.id|broker 用来标识从客户端发送过来的消息，通常被用在日志、度量指标和配额中。|默认值：空字符串|

下面给出几个典型场景的配置示例

场景1：高吞吐量（日志处理）

    fetch.min.bytes=1024           # 每次至少拉取1KB
    fetch.max.wait.ms=500          # 等待最多500ms积累数据
    max.partition.fetch.bytes=10485760  # 单分区拉取10MB
    max.poll.records=1000          # 单次拉取1000条
    enable.auto.commit=true        # 自动提交偏移量 
    auto.commit.interval.ms=10000  # 10秒提交一次

场景2：高可靠性（金融交易）

    enable.auto.commit=false       # 手动提交偏移量
    auto.offset.reset=earliest     # 避免漏消息
    isolation.level=read_committed # 仅消费已提交事务的消息
    max.poll.interval.ms=300000    # 允许处理耗时较长
    session.timeout.ms=60000       # 增大会话超时

场景3：低延迟（实时告警）

    fetch.min.bytes=1             # 立即返回数据
    fetch.max.wait.ms=10          # 最小等待时间
    max.poll.records=100          # 少量消息快速处理
    heartbeat.interval.ms=1000    # 快速检测失效消费者

#### 2.4.5、提交和偏移量

消费者每次调用 `poll()` 时，消费的是 Kafka 中还没有被消费的记录。要做到这一点，就需要记录上一次消费时的消费位移。这个消费位移必须被持久化保存，而不是内存中，否则消费者重启之后就会无法知晓之前的消费位移。

在旧消费者客户端中，消费位移是存储在 ZooKeeper 中的 。 而在新消费者客户端中，消费位移存储在 Kafka 内 部的主题 consumer offsets 中 。 这里把将消费位移存储起来（持久化）的动作称为“提交’，消费者在消费完消息之后需要执行消费位移的提交。

consumer offsets 这个主题会保存每次所发送消息中的分区偏移量，这个主题的主要作用就是消费者触发重平衡后记录偏移使用的，消费者每次向这个主题发送消息，正常情况下不触发重平衡，这个主题是不起作用的，当触发重平衡后，消费者停止工作，每个消费者可能会分到对应的分区，这个主题就是让消费者能够继续处理消息所设置的。


__consumer_offsets 主题里面采用 key 和 value 的方式存储数据。key 是 group.id+topic+分区号，value 就是当前 offset 的值。每隔一段时间，kafka 内部会对这个topic进行 compact，也就是每个 group.id+topic+分区号 就保留最新数据。 

__consumer_offsets 每条消息格式如下

![](https://img2022.cnblogs.com/blog/1110857/202204/1110857-20220413230744937-1888716883.png)

![](https://img2022.cnblogs.com/blog/2591061/202211/2591061-20221116084703852-1262197060.png)

如果提交的偏移量小于客户端最后一次处理的偏移量，那么位于两个偏移量之间的消息就会被重复处理

![](https://camo.githubusercontent.com/dfbf256089e5cbb84ea714f89b6a3746cb64906aadfa9ea750ead6c61322456d/68747470733a2f2f696d67323031382e636e626c6f67732e636f6d2f626c6f672f313531353131312f3230313931312f313531353131312d32303139313132383132343631393931372d3935313334373936342e706e67)

如果提交的偏移量大于最后一次消费时的偏移量，那么处于两个偏移量中间的消息将会丢失

![](https://camo.githubusercontent.com/446552d314a493a8d0c6cdadbe258b1494a0f1f15faf170e9310bc695ffc93ec/68747470733a2f2f696d67323031382e636e626c6f67732e636f6d2f626c6f672f313531353131312f3230313931312f313531353131312d32303139313132383132343632373637332d313538373536383531392e706e67)


##### 自动提交

Kafka 中默认的提交方式为自动提交。自动提交由 enable.auto.commit 控制。enable.auto.commit 被设置为 true，那么每过 5s，消费者会自动把从 `poll()` 方法轮询到的最大偏移量提交上去。提交时间间隔由 auto.commit.interval.ms 控制，默认是 5s。自动提交也是在轮询中进行的。消费者在每次轮询中会检查是否提交该偏移量了，如果是，那么就会提交从上一次轮询中返回的偏移量。

##### 手动提交

自动提交比较简单，是基于时间提交的，难以把握 offset 提交的时机。Kafka 也提供了手动提交的 API。把 auto.commit.offset 设置为 false，可以让应用程序决定何时提交偏移量。

手动提交的方式分为两种：分别是同步提交和异步提交。

1. `commitSync()`（同步提交）：必须等待 offset 提交完毕，再去消费下一批数据。虽然同步提交offset更可靠一些，但是由于其会阻塞当前线程，直到提交成功。因此吞吐量会受到很大的影响。因此更多的情况下，会选用异步提交offset的方式。
2. `commitAsync()`（异步提交）：发送完提交 offset 请求后，就开始消费下一批数据了。

消费者API允许调用 `commitSync()` 和 `commitAsync()` 方法时传入希望提交的 partition 和 offset 的 map，即提交特定的偏移量。

一般情况下，针对偶尔出现的提交失败，不进行重试不会有太大的问题，因为如果提交失败是因为临时问题导致的，那么后续的提交总会有成功的。但是如果在关闭消费者或再均衡前的最后一次提交，就要确保提交成功。


##### 指定 offset 消费

前面介绍过参数 auto.offset.reset，它有三个可取值。
- earliest：自动将偏移量重置为最早的偏移量，--from-beginning。 
- latest（默认值）：自动将偏移量重置为最新偏移量。
- none：如果未找到消费者组的先前偏移量，则向消费者抛出异常。


##### 重复消费和漏消费

> 重复消费：已经消费了数据，没有提交 offset 。

> 漏消费：先提交 offset 后消费，有可能会造成数据的漏消费。


![](https://img2022.cnblogs.com/blog/2591061/202211/2591061-20221117235010389-503479062.png)


本笔记参考以下博客进行总结，若有不对或需要补充地方还请指正，下面附上链接

[Kafka 入门一篇文章就够了](https://github.com/crisxuan/bestJavaer/blob/master/kafka/kafka-basic.md#%E7%94%9F%E4%BA%A7%E8%80%85%E5%88%86%E5%8C%BA%E6%9C%BA%E5%88%B6)

[kafka详解](https://www.cnblogs.com/huangdh/p/16886327.html)

[Kafka底层原理剖析](https://cloud.tencent.com/developer/article/1775065)

[Kafka学习之路](https://www.cnblogs.com/qingyunzong/p/9004509.html)

[Kafka原理篇：图解kakfa架构原理](https://www.51cto.com/article/656518.html)

[深入理解Kafka核心设计及原理(1-6)](https://www.cnblogs.com/zjdxr-up/p/16104558.html)