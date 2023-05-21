---
title: "K8s调度"
description: "K8s调度知识整理。"
date: 2021-06-23T12:18:40+08:00
draft: false
tags: ["go"]
categories: ["go"]
type: "post"
bg_image: "images/backgrounds/page-title.webp"
image: "images/banner/kubernetes.jpg"
---

Kubernetes 中，Pod 通常是容器的载体，主要有如下常见调度方式：

-   `Deployment` 或 `RC`：该调度策略主要功能就是自动部署一个容器应用的多份副本，以及持续监控副本的数量，在集群内始终维持用户指定的副本数量。
-   `NodeSelector`：定向调度，当需要手动指定将 Pod 调度到特定 Node 上，可以通过 Node 的标签（Label）和 Pod 的 nodeSelector 属性相匹配。
-   `NodeAffinity` 亲和性调度：亲和性调度机制极大的扩展了 Pod 的调度能力，目前有两种节点亲和力表达：
    -   requiredDuringSchedulingIgnoredDuringExecution：硬规则，必须满足指定的规则，调度器才可以调度 Pod 至 Node 上（类似 nodeSelector，语法不同）。
    -   preferredDuringSchedulingIgnoredDuringExecution：软规则，优先调度至满足的 Node 的节点，但不强求，多个优先级规则还可以设置权重值。
-   `Taints` 和 `Tolerations`（污点和容忍）：
    -   Taint：使 Node 拒绝特定 Pod 运行；
    -   Toleration：为 Pod 的属性，表示 Pod 能容忍（运行）标注了 Taint 的 Node。

## 参考:

-   [K8S 集群调度原理及调度策略](https://www.cnblogs.com/peng-zone/p/11739433.html#k8s%E8%B0%83%E5%BA%A6%E5%99%A8scheduler)
