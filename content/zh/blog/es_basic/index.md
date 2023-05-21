---
title: "Elasticsearch 知识点"
date: 2022-01-30T17:51:01+08:00
draft: false
# page title background image
bg_image: "images/backgrounds/page-title.jpg"
# about image
image: "images/blog/elasticsearch.png"
# meta description
description: "Elasticsearch 知识点概要总结"
# categories
categories: ["es"]
# tags
tags: ["es"]
# type
type: "post"
---

## 1. Elasticsearch 调优考虑

### 1.1. 设计调优

-   根据业务增量要求，采取基于日期模板创建索引，通过 `roll over API` 滚动索引；
-   使用别名进行索引管理；
-   每天凌晨定时对索引做 `force_merge` 操作，以释放空间；
-   采取冷热分离机制，热数据存储到 `SSD`，提高到检索效率；冷数据定期进行 `shrink` 操作，以缩减存储；
-   采取 `curator` 进行索引的生命周期管理；
-   仅针对需要分词的字段，合理的设置分词器；
-   `Mapping` 阶段充分结合各个字段的属性，是否需要检索，是否需要存储等。

### 1.2. 写入调优

### 1.3. 查询调优

-   禁用 `wildcard`；
-   禁用批量 `terms`（成百上千的场景）；
-   充分利用倒排索引机制，能 keyword 类型尽量用 keyword；
-   数据量打的时候，可以先基于时间敲定索引再检索；
-   设置合理的路由机制。

## 2. Elasticsearch 倒排索引

传统的我们的检索是通过文章，逐个遍历找到对应关键词的位置；而倒排索引，是通过分词策略，形成了词和文章的映射关系表，这种词典+映射表即为倒排索引。有了倒排索引，就能实现 `O(1)` 时间复杂度的效率检索文章了。

倒排索引的底层实现是基于：**`FST(Finite State Transducer)`** 数据结构。

lucene 从 4+版本之后开始大量使用的数据结构是 FST。FST 有两个优点：

-   空间占用小。通过对词典中单词前缀和后缀的重复利用，压缩了存储空间；
-   查询速度快，`O(len(str))`的查询时间复杂度。

## 3. Elasticsearch 是如何实现 Master 选举的？

-   Elasticsearch 的选主是 `ZenDiscovery` 模块负责的，主要包含 `Ping`（节点之间通过这个 `RPC` 来发现彼此）和 `Unicast`（单播模块包含一个主机列表以控制哪些节点需要 ping 通）这两部分；
-   对所有可以成为 master 的节点（`node.master: true`）根据 nodeId 字典排序，每次选举每个节点都把自己所知道节点排一次序，然后选出第一个（第 0 位）节点，暂且认为它是 master 节点。
-   如果对某个节点的投票数达到一定的值（可以成为 master 节点数 `n/2+1`）并且该节点自己也选举自己，那这个节点就是 master。否则重新选举一直到满足上述条件。
-   补充：master 节点的职责主要包括集群、节点和索引的管理，不负责文档级别的管理；data 节点可以关闭 http 功能\*。

## 4. Elasticsearch 的索引数据过多如何处理？

索引数据需要提前规划，

-   **动态索引层面**：基于模板+时间+rollover api 滚动创建索引。
-   **存储层面**：冷热数据分离存储，如可以将 3 天或者近一周内的数据视为热数据，其余为冷数据。对于冷数据不会再写入新数据，可以考虑定期进行 force merge 加 shrink 压缩操作，节省存储空间和检索效率。
-   **部署层面**：如果前期没有规划好，可以通过动态新增机器的方式缓解集群压力，如果主节点规划合理，可以不重启集群进行动态新增。

## 5. Elasticsearch 如何实现 master 选举？

**前置条件**

-   只有候选主节点(`master:true`)的节点才能成为主节点。
-   最小主节点数(`min_master_nodes`)的目的是为了防止脑裂。

Elasticsearch 的选主是 `ZenDiscovery` 模块负责，主要包含 `Ping`（节点之间通过 `RPC` 来发现彼此） 和 `Unicast`（单播模块包含一个主机列表以控制哪些节点需要 `ping` 通） 两部分。
选举的大致流程：

-   第一步：确认候选主节点数达标，`elasticsearch.yml` 设置的值 `dicovery_zen.minimum_master_nodes`
-   第二步：比较：先判断是否具备 master 资格，具备候选主节点资格的优先返回。若两个节点都为候选主节点，则 `id` 小的值会成为主节点，这里的 `id` 为 `string` 类型。

```bash
GET /_cat/nodes?v&h=ip,port,heapPercent,heapMax,id,name
ip port heapPercent heapMax id name
```
