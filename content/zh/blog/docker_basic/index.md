---
title: "Docker 入门知识简要概述"

date: 2022-01-16T17:51:01+08:00
draft: false
# page title background image
bg_image: "images/backgrounds/page-title.jpg"
# about image
image: "images/blog/Docker.png"
# meta description
description: "Docker基础入门的知识"
# categories
categories: ["docker"]
# tags
tags: ["docker"]
# type
type: "post"

aliases: ""
---

## 1.Docker 容器

Docker 容器本质上是宿主机上的进程。Docker 通过 `namespace` 实现了资源隔离，通过 `cgroups` 实现资源限制，通过写时复制机制(`Copy-on-write`)实现了高效的文件操作。

### 1.1. Docker 的资源隔离

**`namespace`** 提供了以下 6 项隔离：

| namespace                            | 系统调用参数  | 隔离内容                   |
| ------------------------------------ | ------------- | -------------------------- |
| Unix Time-sharing System(UTS)        | CLONE_NEWUTS  | 主机和域名                 |
| Unix Interprocess Communication(IPC) | CLONE_NEWIPC  | 信号量、消息队列和共享内存 |
| Process identifier(PID)              | CLONE_NEWPID  | 进程编号                   |
| Network                              | CLONE_NEWNET  | 网络设备、网络栈、端口等   |
| Mount                                | CLONE_NEWNS   | 挂载点(文件系统)           |
| User                                 | CLONE_NEWUSER | 用户和用户组               |

{{<callout note "UTS namespace">}}
{{</callout>}}

分时系统(`Time-sharing System`)中一台主机连接了若干个终端，每个终端有一个用户在使用。分时系统(`Time-sharing System`)中一台主机连接了若干个终端，每个终端有一个用户在使用。

{{<callout note "IPC namespace">}}
{{</callout>}}

Linux 进程间通信由 **`System V IPC`**，基于 Socket 的 IPC 和 POSIX IPC 发展而来，主要的通信手段有:

-   管道(`Pipe`)
    -   无名管道 - 只能用于具有亲缘关系的进程之间的通信（父子进程或者兄弟进程之间）
        半双工的通信模式，具有固定的读端和写端
        是一种特殊的文件，不属于其他任何文件系统并且只存在于内存中
    -   有名管道(`FIFO`)
        使互不相关的两个进程间实现彼此通信
        可以通过路径名来指出，并且在文件系统中是可见的。在建立了管道之后，两个进程就可以把它当做普通文件一样进行读写操作
        先进先出规则
-   信号(`Signal`)：用于通知接受进程有某种事件发生，除了用于进程间通信外，进程还可以发送信号给进程本身
-   消息队列(Message Queue)
    有足够权限的进程可以向队列中添加消息，被赋予读权限的进程则可以读走队列中的消息。
    克服了信号承载信息量少，管道只能承载无格式字节流（要求管道的读出方和写入方必须事先约定好数据的格式，比如多少字节算作一个消息等）以及缓冲区大小受限等缺点。
-   共享内存
    使得多个进程可以访问同一块内存空间。
-   信号量(`semaphore`)
    主要用于进程间以及同一进程不同线程之间的同步。
-   套接字(`Socket`): 可用于不同机器之间的进程间通信，必须包含
    -   地址，由 ip 与 端口组成，像 192.168.0.1:80。
    -   协议，socket 所用的传输协议 TCP、UDP、raw IP。

PC 资源包括信号量、消息队列和共享内存。通过 IPC namespace 可以实现容器与宿主机、容器与容器之间的 IPC 隔离。

{{<callout note "PID namespace">}}
{{</callout>}}

**`PID namespace`** 隔离对进程 PID 重新编号，两个不同 namespace 下的进程没有关系，因此 PID 也可以相同。内核为所有的 PID namespace 维护了一个树状结构。

{{<callout note "Mount namespace">}}
{{</callout>}}

**`mount namespace`** 通过隔离文件系统挂载点隔离文件系统，它是第一个 Linux namespace，标识位为 `CLONE_NEWNS`。隔离之后不同的 `mount namespace` 中的文件结构互不影响。

{{<callout note "Network namespace">}}
{{</callout>}}

**`network namespace`** 提供了关于网络资源的隔离，包括网络设备、IPv4 和 IPv6 协议栈、IP 路由表、防火墙、/proc/net 目录、/sys/class/net 目录、套接字(socket)等。

{{<callout note "User namespace">}}
{{</callout>}}

**`user namespace`** 主要隔离了安全相关的标识符和属性(用户 ID、用户组 ID、root 目录、key(密钥)、特殊权限)。

### 1.2. Docker 的资源限制

**`cgroups`** 是 Linux 内核提供的一种机制。

这种机制可以根据需求把一系列系统任务及其子任务整合(或分隔)到按资源划分等级的不同组内，限制了被 `namespace` 隔离起来的资源，并为资源设置权重、计算使用量(`CPU, Memory, IO` 等)、操控任务启停等。

从而为系统资源管理提供了一个统一的框架。

从本质上来说，cgroups 是内核附加在程序上的一系列钩子(Hook),通过程序运行时对资源的调度触发相应的钩子以达到资源追踪和限制的目的。

**`cgroup`** 提供的主要功能如下：

-   资源限制：限制任务使用的资源总额，并在超过这个配额时发出提示
-   优先级分配：分配 CPU 时间片数量及磁盘 IO 带宽大小、控制任务运行的优先级
-   资源统计：统计系统资源使用量，如 CPU 使用时长、内存用量等
-   任务控制：对任务执行挂起、恢复等操作
