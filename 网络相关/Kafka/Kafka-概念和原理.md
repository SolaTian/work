# Kafka

## 1、Kafka基本概念

### 1.1、什么是Kafka

> Kafka是一种分布式发布-订阅消息系统，主要用途就是实时数据流的处理和传输。

那么有个问题，什么是**消息系统**？

### 1.1.1、消息系统

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

### 1.1.2、发布-订阅式与点对点式

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
|`Replica`|`Replica`是`Partition`的副本，用于实现数据的冗余和容错。每个`Partition`都有一个`Leader`副本和多个`Follower`副本。`Leader`副本负责处理读写请求(生产者发送数据的对象，消费者消费数据的对象)，`Follower`副本从`Leader`副本复制数据以保持数据一致性|当`Leader`副本出现故障时，`Kafka`会自动从`Follower`副本中选举出新的`Leader`副本，以保证服务的连续性|
|`Zookeeper`|`Zookeeper`不是`Kafka`的直接组成部分，但它在`Kafka`集群中扮演着至关重要的角色，管理`Kafka`集群的元数据（如`Broker`信息、`Topic`信息、`Partition`信息等），并协调`Broker`之间的交互和选举。|`Zookeeper`的高可用性和强一致性保证了`Kafka`集群的稳定性和可靠性|


#### 1.2.1、Topic、Consumer Group、Partition 的关系

> Partition 数量决定了每个 Consumer Group 中并发消费者的最大数量。

![](https://ask.qcloudimg.com/http-save/2039230/ev6txob5p8.png)

如上面左图所示，如果只有两个分区，即使一个组内的消费者有4个，也会有两个空闲的。   
如上面右图所示，有4个分区，每个消费者消费一个分区，并发量达到最大4。


![](https://ask.qcloudimg.com/http-save/2039230/j1r2rsufsw.png)

不同的消费者组消费同一个 Topic，这个 Topic 有4个分区，分布在两个节点上。左边的 Group1 有2个消费者，每个消费者就要消费两个分区才能把消息完整的消费完，右边的 Group2 有4个消费者，每个消费者消费一个分区即可.

总结这三者的关系：
1. 某一个主题下的分区数，对于消费该主题的同一个消费组下的消费者数量，应该小于等于该主题下的分区数。如：某一个主题有4个分区，那么消费组中的消费者应该小于等于4，而且最好与分区数成整数倍 1、2、4 。
2. 同一个分区下的数据，在同一时刻，不能同一个消费组的不同消费者消费。
3. 分区数越多，同一时间可以有越多的消费者来进行消费，消费数据的速度就会越快，提高消费的性能

#### 1.2.2、Segment 文件

##### .index 文件和 .log 文件
一个 Partition 当中由多个 Segment 文件组成，每个 Segment 文件，包含两部分，一个是 .log 文件，另外一个是 .index 文件，其中 .log 文件包含了发送的数据存储，.index 文件，记录的是 .log 文件的数据索引值，以便于加快数据的查询速度。

![.log 和 .index 文件](https://ask.qcloudimg.com/http-save/2039230/002wqqdvqv.png)

如上图，左半部分是索引文件 .index ，里面存储的是一对一对的 key-value ，其中 key 是消息在数据文件（对应的 .log 文件）中的编号，比如“1,3,6,8……”，分别表示在log文件中的第1条消息、第3条消息、第6条消息、第8条消息……

比如索引文件中 3,497 代表：数据文件中的第三个 Message，它的偏移地址为497。再来看数据文件中，Message 368772表示：在全局 Partiton 中是第368772个 Message。

index 文件中并没有为数据文件中的每条消息都建立索引，而是采用了稀疏存储的方式，每隔一定字节的数据建立一条索引。这样避免了索引文件占用过多的空间，从而可以将索引文件保留在内存中，通过内存映射（mmap）直接进行内存映射。但缺点是没有建立索引的 Message 也不能一次定位到其在数据文件的位置，从而需要做一次顺序扫描，但是这次顺序扫描的范围就很小了。

##### segment文件命名规则
Partition 全局的第一个 Segment 从0开始，后续每个 Segment 文件名为上一个全局 Partition 的最大 offset（偏移 Message 数）。数值最大为64位long大小，20位数字字符长度，没有数字就用 0 填充。

通过索引信息可以快速定位到 Message。通过 .index 元数据全部映射到内存，可以避免 Segment File的 IO 磁盘操作；通过索引文件稀疏存储，可以大幅降低 .index 文件元数据占用空间大小。

> 稀疏索引：为了数据创建索引，但范围并不是为每一条创建，而是为某一个区间创建；

- 优点：就是可以减少索引值的数量。
- 缺点：找到索引区间之后，要得进行第二次处理。


#### 1.2.3、Message 的结构

生产者发送到 Kafka 的消息，都被 Kafka 包装成了 Message。

![Message的物理结构](https://ask.qcloudimg.com/http-save/2039230/5q1z5tmizh.png)

每条消息都被包装成上图这个结构，只有最后一个字段才是真正生产者发送的消息数据。

#### 1.2.4、Kafka 分区副本

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

要真正吃透`Kafka`的底层原理很不容易。这里也只是大概的介绍一下`Kafka`的大致工作原理。

除了之前提到的`Kafka`的组成，在介绍原理之前，还有几个术语需要介绍一下
- 消息偏移量`offset`：表示分区`Partition`中每条消息的位置信息，是一个单调递增且不变的值。

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


### 2.2、Kafka 结点的领导者——Controller

> Controller：是 Kafka 集群中的一个特殊的 Broker，它除了像普通 Broker那样对外提供消息的生产、消费、同步功能外，还额外承担了管理 Kafka 集群的 Broker、Topic、分区等职责。

Controller 负责管理这些元数据，那么它与 Zookeeper 的元数据管理有什么区别？

- Zookeeper：核心职责是可靠存储元数据和通知变更（通过 Watcher ）。它是被动的存储系统，不参与决策。
- Controller：作为集群的“大脑”，主动管理状态（如故障恢复、副本均衡）和执行决策（如 Leader 选举），并将结果持久化到 Zookeeper。

在后续新版的 Kafka 中，使用 KRaft 代替了 Zookeeper，实现了元数据管理的去中心化和性能优化。


#### 2.2.1、Controller 的选举过程

`Kafka`集群中的每一个`Broker`都有可能成为`Controller`，具体的选举依赖于`Zookeeper`。

选举流程：

1. 候选者注册：在`Kafka`集群启动时，每个`Broker`都有可能成为`Controller`的候选者。各候选者会尝试在`Zookeeper`中创建临时有序临时节点（通常是`/controller`）来表明自己的参与。这些节点是临时的，意味着当`Broker`宕机或退出集群时，相应的节点会被自动删除。
  
2. 节点的排序与选举：`Zookeeper`对候选者结点进行排序，具有最小序号的节点成为新的`Controller`，如果多个候选者具有相同的最小序号，那么`Zookeeper`会根据节点的创建时间来选择最终的`Controller`，采用的是“先到先得”的抢占模式，`Zookeeper`会保证有且仅有一个`Broker`能创建成功，这个`Broker`就会成为集群的总控器`Controller`成功创建节点的 `Broker`将自己的`ID`、版本号等信息写入`/controller`节点。例如，节点内容可能为：`{"version":1, "brokerid":1001, "timestamp":"..."}`；

3. 选举完成：一旦选举完成，新的`Controller`节点将被选出，并且其他候选者`Broker`将知道哪个节点成为了新的`Controller`(监听机制)。新的`Controller`节点将负责管理`Kafka`集群的状态、执行分区分配、`Leader`选举等操作，会向`Zookeeper`中写入`epoch`值，即`Controller`的版本号，其余`Broker`在接收元数据变更时，会校验该版本号，拒绝旧的`Controller`的指令；

4. 故障转移和重新选举：当前的`Controller`节点发生故障或失效时，原先的`/controller`结点失效，`Kafka`集群会自动触发`Controller`的重新选举过程。这个过程由`Zookeeper`的临时节点和节点监听机制来保证。新的`Controller`候选者将尝试在`Zookeeper`中创建临时有序节点，参与新一轮的`Controller`选举过程。`ZooKeeper`将重新处理候选者节点，选举出新的`Controller`节点并更新`epoch`值，并接管集群管理任务，确保`Kafka`集群的正常运行和高可用性。


#### 2.2.2、Controller的职责

1. 集群的状态管理：
   1. 为`Zookeeper`中的`/brokers/ids/`节点添加`BrokerChangeListener`，用来处理`Broker`增减的变化。监控集群中`Broker`的增减变化，包括处理`Broker`的加入，主动关闭和宕机，`Controller`需要及时更新集群元数据，并将集群变化通知到所有的`Broker`集群节点；
   2. 分区与副本管理：`Controller`负责管理和监控集群中所有分区的状态，包括`Topic`分区的创建、删除、状态转换以及副本的选举和状态更新。它通过`ZooKeeper`来协调这些任务，确保分区和副本的高可用性和一致性;

2. 元数据管理：`Controller`还负责维护集群的元数据信息，从`Zookeeper`中读取和更新集群的元数据信息，如`Topic`的分区信息、每个分区的`Leader`副本信息。当集群中的元数据发生变化时，`Controller`会及时更新集群元数据并将更新后的信息同步给集群中的所有`Broker`，确保每个`Broker`都能获取到最新的元数据信息；

3. 故障切换与内容复制：当分区的`Leader`副本发生故障时，`Controller`负责进行故障切换，选举新的`Leader`副本，并确保数据的正确复制和同步；



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

#### 2.2.3、Leader 的选举

由于 Controller 监听了很多的 Zookeeper 结点，所以能够感知到 Broker 的存活。


> 副本优先机制：当分区 Leader 所在的 Broker 挂掉之后，Controller 会从每个 Partition 的 ISR 中取出第一个 Broker 里面的 Follower 作为 Leader。


### 2.3、生产者 Producer

正常的生产逻辑：
- 配置生产者客户端参数和创建相应的生产者实例
- 构建发送的消息
- 发送消息
- 关闭生产者实例

#### 2.3.1、数据生产流程

![数据生产流程](https://img2022.cnblogs.com/blog/2742789/202202/2742789-20220228223715656-1782209181.png)

写入一条数据，需要指定四个参数：Topic、Partition、Key 和 Value，其中 Topic 和 Value (要写入的数据)是必须要指定的，而 Key 和 Partition 是可选的。对于一条记录，先对其进行序列化，然后根据 Topic 和 Partition，放进对应的发送队列中。如果 Partition 没填，分为两种情况：
- Key 有值，按照 Key 进行哈希，相同 Key 去一个Partition
- Key 无值，轮循选出 Partition

Producer 将会和 Topic 下所有 Partition Leader 保持 socket 连接，消息由 Producer 直接通过socket 发送到 Broker。其中 Partition Leader的位置注册在Zookeeper中，Producer作为Zookeeper Client，已经注册了 watch 用来监听 Partition Leader 的变更事件，因此，可以准确的知道谁是当前的 leader。

#### 2.3.2、重要生产者参数

|生产者参数|说明|
|-|-|
|acks|指定分区中必须要有多少个副本接受这条消息，生产者才被认为写成功。acks 参数有3种类型的值(都是字符串类型)：<li> acks="1" (默认)。生产者发送消息之后，只要分区的 leader 副本成功写入消息，那么它就会收到来自服务端的成功响应。 折中方案。消息写入 leader 副本并 返回成功响应给生产者，且在被其他 follower 副本拉取之前 leader 副本崩溃，那么此 时消息还是会丢失<li>acks="0" 。生产者发送消息之后不需要等待任何服务端的响应。<li>acks="-1/all"。生产者在消息发送之后，需要等待 ISR 中的所有副本都成功 写入消息之后才能够收到来自服务端的成功响应。(最高可靠,leader 宕机也不丢失，生产者收到异常告知此次发送失败)|
|max.request.size|限制生产者客户端能发送的消息的最大值。默认 1M|
|compression.type|指定消息压缩方式，默认 "none"，还可以配置为 "gzip", "snappy", "lz4"|
|connections.max.idle.ms|多久关闭闲置的连接|
|retries, retry.backoff.ms|生产者重试次数和间隔。在需要保证消息顺序的场合建议把参数 max.in.flight . requests .per.connection 配置为 1|
|linger.ms|指定生产者发送 ProducerBatch 之前等待更多消息 (ProducerRecord) 加入 ProducerBatch 的时间<li>生产者客户端会在 ProducerBatch 被填满或等待时间超过 linger .ms 值时发迭出去。增大这个参数的值会增加消息的延迟，但是同时能提升一定的吞 吐量。|
|receive.buffer.bytes|socket 接受消息缓冲区(SO_RECBUF) 大小，默认 32kb。设置-1 则使用操作系统默认值|
|request.timeout.ms|这个参数用来配置 Producer等待请求响应的最长时间，默认值为 30000 (ms)|


####  2.3.2、同步发送和异步发送

生产者给 Kafka 发送数据，可以采用同步方式或者异步方式，默认异步方式，可通过 producer.type 属性进行配置，Kafka 通过配置 request.required.acks 属性来确认消息的生产。

ack

> 同步方式：发送一批数据给 Kafka 后，等待 Kafka 返回结果：
- 生产者等待10s，如果 broker 没有给出 ack 响应，就认为失败。
- 生产者重试3次，如果还没有响应，就报错.

> 异步方式：发送一批数据给 Kafka，只是提供一个回调函数：
- 先将数据保存在生产者端的 buffer 中。buffer 大小是2万条。
- 满足数据阈值或者数量阈值其中的一个条件就可以发送数据。
- 发送一批数据的大小是500条。

注：如果broker迟迟不给ack，而buffer又满了，开发者可以设置是否直接清空buffer中的数据。

### 数据消费流程






本笔记参考以下博客：

[Kafka底层原理剖析](https://cloud.tencent.com/developer/article/1775065)

[Kafka学习之路](https://www.cnblogs.com/qingyunzong/p/9004509.html)

[Kafka原理篇：图解kakfa架构原理](https://www.51cto.com/article/656518.html)