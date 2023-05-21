---
title: "Kafka知识点"
description: "Kafka知识点概要总结。"
date: 2022-01-20T12:18:40+08:00
draft: false
tags: ["kafka"]
categories: ["kafka"]
type: "post"
bg_image: "images/backgrounds/page-title.webp"
image: "images/blog/kafka.png"
---

## 1. Apache Kafka 是什么？

Apache Kafka 是一款分布式流处理平台，用于实时构建流处理应用。它有一个核心的功能广为人知，即作为企业级的消息引擎被广泛使用（通常也会称为消息总线 message bus）。

Kafka 主要有两大应用场景：

-   消息队列：建立实时流数据管道，以可靠地在系统或应用程序之间获取数据。
-   数据处理：构建实时的流数据处理程序来转换或处理数据流。

## 2. Kafka 的设计是什么样的？

Kafka 将消息以 topic 为单位进行归纳，将向 Kafka topic 发布消息的程序称为生产者 producer；将订阅 topics 并消费消息的程序称为消费者 consumer。

Kafka 以集群的方式运行，可以由一个或多个服务组成，每个服务叫做一个 broker。producer 通过网络将消息发送到 Kafka 集群，集群向消费者提供消息。

## 3. Kafka 消息是采用 Pull 模式，还是 Push 模式？

生产者使用 push 模式将消息发布到 Broker，消费者使用 pull 模式从 Broker 订阅消息。

push 模式很难适用消费速率不同的消费者，如果 push 的速度太快，容易造成消费者拒绝服务或网络拥塞；如果 push 的速度太慢，容易造成消费者性能浪费。但是采用 pull 的方式也有一个缺点，
就是当 Broker 没有消息时，消费者会陷入不断地轮训中，为了避免这点，kafka 有个参数可以让消费者阻塞知道是否有新消息到达。

## 4. Kafka 与传统消息系统之间的区别

-   Kafka 持久化日志，这些日志可以被重复读取和无限期保留
-   Kafka 是一个分布式系统：它以集群的方式运行，可以灵活伸缩，在内部通过复制数据提升容错能力和高可用性
-   Kafka 支持实时的流式处理

## 5. 在 Kafka 中，Zookeeper 的作用是什么？

目前，Kafka 使用 Zookeeper `存放集群元数据、成员管理、Controller 选举`，以及其他一些管理类任务。之后，等 KIP-500 提案完成后，Kafka 将完全不再依赖于 Zookeeper。

-   “存放元数据”是指主题分区的所有数据都保存在 Zookeeper 中，且以它保存的数据为权威，其他人都要与它保持对其。
-   “成员管理”是指 Broker 节点的注册、注销以及属性变更，等等。
-   “Controller 选举”是指选举集群 Controller，而其他管理类任务包括但不限于主题删除、参数配置等。

**KIP-500 思想**，是使用社区自研的基于 Raft 的共识算法，替代 Zookeeper，实现 Controller 自选举。

## 6. Kafka 中的位移（offset）作用

在 Kafka 中，每个主题分区下的每条消息都被赋予了一个唯一的 ID 数值，用于标识他在分区中的位置。这个 ID 数值，就被称为位移，或者叫做偏移量。
一旦消息被写入到分区日志，他的偏移量将不能被修改。

## 7. Kafka 为什么那么快？（高性能是如何保证的）

### 7.1. 顺序读写

Kafka 使用了磁盘顺序读写来提升性能。Kafka 的 `message` 是不断追加到本地磁盘文件的末尾，而不是随机写入。这种方式有一个缺陷——无法删除数据，
所以 Kafka 是不会删除数据的，它会把所有的数据都保留下来，每个消费者对每个 topic 都有一个 offset 来表示读取到了第几条数据。Kafka 提供了两种策略来删除数据，一是基于时间，
二是基于 partition 文件大小。

### 7.2. PageCache

Kafka 利用操作系统本身的 `Page Cache`，就是利用操作系统自身的内存而不是 JVM 空间内存。这样做的好处有：

-   避免 Object 消耗：如果是 Java 堆，Java 对象的内存消耗比较大，通常是所存储数据的两倍甚至更多。
-   避免 GC 问题：随着 JVM 中数据不断增多，垃圾回收将会变得复杂与缓慢，使用系统缓存就不会存在 GC 问题。

通过操作系统的 `Page Cache`，Kafka 的读写操作基本上是基于内存的，读写速度得到了极大的提升

### 7.3. 零拷贝

零拷贝并非指一次拷贝都没有，而是避免了在内核空间和用户空间之间的拷贝。

### 7.4. 分区分段+索引

Kafka 的 `message` 是按照 topic 分类存储的，topic 中的数据又是按照一个一个的 partition 即分区存储到不同的 broker 节点。每个 `partition` 对应了
操作系统上的一个文件夹，`partition` 实际上又是按照 segment 分段存储的。这也非常符合分布式系统区分桶的设计思想。

### 7.5. 批量读写

Kafka 在写入数据时，可以启用批次写入，这样可以避免在网络上频繁传输单个消息带来的延迟和带宽开销。

### 7.6. 批量压缩

Kafka 使用了批量压缩，即将多个消息一起压缩而不是单个消息压缩；Kafka 允许使用递归的消息集合，批量的消息可以通过压缩的形式传输并在日志中也可以保持压缩格式，知道被消费者解压缩；
Kafka 支持多种压缩协议，包括 `Gzip` 和 `Snappy` 压缩协议

## 8. Kafka producer 发送数据，ack 为 0，1，-1 分别表示什么？

-   `1（默认）`：数据发送到 Kafka 后，经过 leader 成功接收消息的确认，就算是发送成功了。在这种情况下，如果 leader 宕机了，则会丢失数据。
-   `0`：生产者将数据发送出去就不管了，不去等到任何返回。这种情况下数据传输效率最高，但是数据可靠性却是最低的。
-   `-1`：producer 需要等到 `ISR` 中所有 follower 都确认接收到数据后才算一次发送完成，可靠性最高，当 `ISR` 中所有 `Replica` 都向 `Leader` 发送 ACK 时，leader 才 commit，这时候 producer 才能认为一个请求中的消息都 commit 了。

## 9. Kafka 如何保证消息不丢失?

首先需要弄明白消息为什么会丢失，对于一个消息队列，会有 **生产者、MQ、消费者** 这三个角色，在这三个角色数据处理和传输过程中，都有可能会出现消息丢失。

![数据处理和传输过程](kafka-msg-ensure.png)

消息丢失的原因以及解决办法：

### 9.1. 消费者异常导致的消息丢失

消费者可能导致数据丢失的情况是：消费者获取到了这条消息后，还未处理，`Kafka` 就自动提交了 `offset`，这时 `Kafka` 就认为消费者已经处理完这条消息，其实消费者才刚准备处理这条消息，这时如果消费者宕机，那这条消息就丢失了。

消费者引起消息丢失的主要原因就是消息还未处理完 `Kafka` 会自动提交了 `offset`，那么只要关闭自动提交 `offset`，消费者在处理完之后手动提交 `offset`，就可以保证消息不会丢失。但是此时需要注意重复消费问题，比如消费者刚处理完，还没提交 `offset`，这时自己宕机了，此时这条消息肯定会被重复消费一次，这就需要消费者根据实际情况保证幂等性。

### 9.2. 生产者数据传输导致的消息丢失

对于生产者数据传输导致的数据丢失主常见情况是生产者发送消息给 `Kafka`，由于网络等原因导致消息丢失，对于这种情况也是通过在 **producer** 端设置 **acks=all** 来处理，这个参数是要求 `leader` 接收到消息后，需要等到所有的 `follower` 都同步到了消息之后，才认为本次写成功了。如果没满足这个条件，生产者会自动不断的重试。

### 9.3. Kafka 导致的消息丢失

Kafka 导致的数据丢失一个常见的场景就是 `Kafka` 某个 `broker` 宕机，，而这个节点正好是某个 `partition` 的 `leader` 节点，这时需要重新重新选举该 `partition` 的 `leader`。如果该 `partition` 的 `leader` 在宕机时刚好还有些数据没有同步到 follower，此时 `leader` 挂了，在选举某个 `follower` 成 `leader` 之后，就会丢失一部分数据。

对于这个问题，Kafka 可以设置如下 4 个参数，来尽量避免消息丢失：

-   给 `topic` 设置 `replication.factor` 参数：这个值必须大于 `1`，要求每个 `partition` 必须有至少 `2` 个副本；
-   在 `Kafka` 服务端设置 `min.insync.replicas` 参数：这个值必须大于 `1`，这个参数的含义是一个 `leader` 至少感知到有至少一个 `follower` 还跟自己保持联系，没掉队，这样才能确保 `leader` 挂了还有一个 `follower` 节点。
-   在 `producer` 端设置 `acks=all`，这个是要求每条数据，必须是写入所有 `replica` 之后，才能认为是写成功了；
-   在 `producer` 端设置 `retries=MAX`（很大很大很大的一个值，无限次重试的意思）：这个参数的含义是一旦写入失败，就无限重试，卡在这里了。

## 10. Kafka 如何保证消息的顺序性？

在某些业务场景下，我们需要保证对于有逻辑关联的多条 MQ 消息被按顺序处理，比如对于某一条数据，正常处理顺序是`新增->更新->删除`，最终结果是数据被删除；如果消息没有被按序消费，处理顺序
可能是`删除->新增->更新`，最终数据没有被删除掉，可能会产生一系列的逻辑错误。对于如何保证消息的顺序性，主要需要考虑如下两点：

-   如何保证消息在 kafka 中的顺序性；
-   如何保证消费者处理消费的顺序性。

### 10.1. 如何保证消息在 Kafka 中的顺序性？

对于 Kafka，如果我们创建了一个 topic，默认有三个 `Partition`，生产者在写数据的时候，可以指定一个 key，比如在订单 `topic` 中我们可以指定订单 id 作为 key，那么相同订单 id 的数据，一定会被分发到同一个 `Partition` 中去，
而且这个 `Partition` 中的数据一定是有顺序的。消费者从 `Partition` 中取出来数据的时候，也一定是有顺序的。通过指定 key 的方式首先可以保证在 kakfa 内部消息是有序的。

### 10.2. 如何保证消费者处理消息的顺序性？

对于某个 `topic` 的一个 `Partition`，只能被同组内部的一个 `consumer` 消费，如果这个 `consumer` 内部还是单线程处理，那么其实只要保证消息在 MQ 内部是有顺序的就可以保证消费也是有顺序的。但是单线程吞吐量太低，在处理大量 MQ 消息时，
我们一般会开启多线程消费机制，那么如何保证消息在多个线程之间是被顺序处理的呢？对于多线程消费我们可以预先设置 `N` 个内存 `Queue`，具有相同 `key` 的数据都放到同一个内存 `Queue` 中；然后开启 `N` 个线程，每个线程分别消费一个内存 `Queue` 的
数据即可，这样就能保证顺序性。当然，消息放到内存 `Queue` 中，有可能还未被处理，`consumer` 发生宕机，内存 `Queue` 中的数据会全部丢失，这就转变为上面提到的**如何保证消息的可靠传输**的问题了。

## 11. Kafka 中的 ISR、AR 分别代表什么？ISR 的伸缩指什么？

-   `ISR`：In-Sync Replicas 副本同步队列
-   `AR`：Assigned Replicas 所有副本

`ISR` 是由 leader 维护，follower 从 leader 同步数据有一些延迟（包括`延迟时间replica.lag.time.max.ms`和`延迟条数 replica.lag.max.messages`两个维度，
当前最新的版本 0.10.x 中只支持`replica.lag.time.max.ms`这个维度），任意一个超过阈值都会把 follower 剔除出 `ISR`，存入 `OSR` 列表，新加入的 follower 也会先存放在 `OSR` 中。

```
AR = ISR + OSR
```

## 12. 分区 Leader 选举策略有几种？

分区的 Leader 副本选举对用户是完全透明的，它是由 `Controller` 独立完成的。

-   **OfflinePartition Leader 选举**： 每当有分区上线时，就需要执行 Leader 选举。所谓的分区上线，可能是创建了新的分区，也可能是之前的下线分区重新上线。这是最常见的分区 Leader 选举场景。
-   **ReassingPartition Leader 选举**：当手动运行`kafka-reassign-partitions`命令，或者是调用 Admin 的 `alterPartitionReassignments` 方法执行分区副本重分配时，可能触发此类选举。假设
    原来的 AR 是 [1, 2, 3]，Leader 是 1，当执行副本重分配后，副本集合 AR 被设置为 [4, 5, 6]，显然，Leader 必须要变更，此时会发生`Reassign Partition Leader`选举。
-   **PreferredReplicaPartition Leader 选举**：当手动运行 `kafka-preferred-replica-election` 命令，或自动触发了 `Preferred Leader 选举`时，该类策略被激活。所谓的 `Preferred Leader`，指的是
    AR 中的第一个副本。比如 AR 是[3, 2, 1]，那么 `Preferred Leader` 就是 3。
-   **ControllerShutdownPartition Leader 选举**：当 Broker 正常关闭时，该 Broker 上的所有 Leader 副本都会下线，因此，需要为受影响的分区执行相应的 Leader 选举。

## 13. 为什么 kafka 不支持读写分离？

在 kafka 中，生产者写入消息、消费者读取消息的操作都是与 leader 副本进行交互的，从而实现的是一种主写主读的生产消费模型。

Kafka 并不支持主写从读，因为主写从读有 2 个明显的缺点：

-   **数据一致性问题**。数据从主节点转到从节点必然会有一个延时的时间窗口，这个时间窗口会导致主从节点之间的数据不一致。某一时刻，在主节点和从节点
    中 A 数据的值都为 `X`，之后将主节点中 A 的值修改为 `Y`，那么在这个变更通知到从节点之前，应用读取从接待中的 A 数据的值并不为最新的 `Y`，由此便产生了数据不一致的问题。
-   **延时问题**。类似 Redis 这类中间件，数据从写入主节点到同步至从节点中的过程需要经历`网络->主节点内存->网络->从节点内存`这几个阶段，整个过程会耗费一定的时间。而在
    Kafka 中，主从节点会比 Redis 更加耗时，他需要经历`网络->主节点内存->主节点磁盘->网络->从节点内存->从节点磁盘`这几个阶段。对延时敏感的应用而言，主写从读的功能并不太适用。

## 参考

-   https://github.com/flycash/interview-baguwen/blob/main/mq/Kafka.md