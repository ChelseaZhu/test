### 集群配置

#### Broker端配置参数

- ##### 与存储相关参数：

    `log.dirs`和`log.dir`。只对前一个参数进行配置即可，在线上生产环境中一定要为log.dirs配置多个路径，具体格式是一个 CSV 格式，也就是用逗号分隔的多个路径，比如`/home/kafka1,/home/kafka2,/home/kafka3`这样。如果有条件的话你最好保证这些目录挂载到不同的物理磁盘上。

    `提升读写性能：`比起单块磁盘，多块物理磁盘同时读写数据有更高的吞吐量。

    `能够实现故障转移：`即 Failover。这是 Kafka 1.1 版本新引入的强大功能。要知道在以前，只要 Kafka Broker 使用的任何一块磁盘挂掉了，整个 Broker 进程都会关闭。但是自 1.1 开始，这种情况被修正了，坏掉的磁盘上的数据会自动地转移到其他正常的磁盘上，而且 Broker 还能正常工作。还记得上一期我们关于 Kafka 是否需要使用 RAID 的讨论吗？这个改进正是我们舍弃 RAID 方案的基础：没有这种 Failover 的话，我们只能依靠 RAID 来提供保障。

- ##### 与Zookeeper相关参数：

    `zookeeper.connect`。这也是一个 CSV 格式的参数，比如我可以指定它的值为zk1:2181,zk2:2181,zk3:2181。2181 是 ZooKeeper 的默认端口。

- ##### 与Broker连接相关参数：

    `listeners：`学名叫监听器，其实就是告诉外部连接者要通过什么协议访问指定主机名和端口开放的 Kafka 服务。

    `advertised.listeners：`和 listeners 相比多了个 advertised。Advertised 的含义表示宣称的、公布的，就是说这组监听器是 Broker 用于对外发布的。

- ##### 关于Topic管理相关参数：

    `auto.create.topics.enable：`是否允许自动创建 Topic。最好将这个参数设置城false，设置成true很有可能因为你向一个不存在的topic发送消息而将这个topic创建了，这种情况最好规避。

    `unclean.leader.election.enable：`是否允许 Unclean Leader 选举。如果设置成 false，那么就坚持之前的原则，坚决不能让那些落后太多的副本竞选 Leader。这样做的后果是这个分区就不可用了，因为`没有 Leader 了`。反之如果是 true，那么 Kafka 允许你从那些“跑得慢”的副本中选一个出来当 Leader。这样做的后果是`数据有可能就丢失了`，因为这些副本保存的数据本来就不全，当了 Leader 之后它本人就变得膨胀了，认为自己的数据才是权威的。

    `auto.leader.rebalance.enable：`是否允许定期进行 Leader 选举。

- ##### 关于数据留存相关参数

    `log.retention.{hours|minutes|ms}：`这是个“三兄弟”，都是控制一条消息数据被保存多长时间。从优先级上来说 ms 设置最高、minutes 次之、hours 最低。比如log.retention.hours=168表示默认`保存 7 天的数据`，自动删除 7 天前的数据。很多公司把 Kafka 当做存储来使用，那么这个值就要相应地调大。

    `log.retention.bytes：`这是指定 Broker 为消息保存的总磁盘容量大小。这个值`默认是 -1`，表明你想在这台 Broker 上保存多少数据都可以，至少在容量方面 Broker 绝对为你开绿灯，不会做任何阻拦。

    `message.max.bytes：`控制 Broker 能够接收的最大消息大小。`默认的 1000012 `太少了，还不到 1MB。建议将这个值设置大一点。

#### Topic级别参数

`retention.ms：`规定了该 Topic 消息被保存的时长。默认是 7 天，即该 Topic 只保存最近 7 天的消息。一旦设置了这个值，它会覆盖掉 Broker 端的全局参数值。

`retention.bytes：`规定了要为该 Topic 预留多大的磁盘空间。和全局参数作用相似，这个值通常在多租户的 Kafka 集群中会有用武之地。当前默认值是 -1，表示可以无限使用磁盘空间。

`max.message.bytes：`它决定了 Kafka Broker 能够正常接收该 Topic 的最大消息大小。

> 提示：Topic级别参数优先级是大于Broker全局参数的

#### 
