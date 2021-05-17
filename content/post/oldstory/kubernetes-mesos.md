---
title: "kubernetes vs mesos"
date: 2021-05-17
draft: false
---

当kubernetes和mesos这两个词同时出现时，可能很多人仿佛看到了一个成功者与失败者站在一起，接下来可能会列举一堆mesos做不到，但是k8s完美解决的栗子。我们也会看到， 分别把kubernetes和mesos当作集中式调度和二层调度的代表，对比一通，作出前者更适应现代IT架构的结论。写这篇文章是想稍微消解对kubernetes的崇拜，提供一个更客观的评价这两种技术的角度。 


拿mesos和kubernetes比较是不公正的。 Mesos 论文开篇第一句就指出，mesos是一个让不同计算框架共享集群资源的框架， 它关注的核心指标是资源利用率。而kubernetes是一个容器编排框架，当然容器编排包含容器调度。 kubernetes的野心显然更大，它提供一套强有力的语言，去描述PaaS，从而让PaaS可编程化。mesos和kubernetes有交集，但是不能认为kubernetes是在同一个问题上更好的解决方案。 Mesos为了让不同计算框架共享集群，提出了二层调度方案， 让上层的计算框架调度任务，而mesos本身对资源进行抽象并进行资源层面的调度，更适合短任务调度或者批量任务，比如Hadoop。kubernetes 是个更宏大的事情，他提供一套语言，去描述资源、描述工作负载等等，希望所有的service能跑在他上面，所以他的设计实现首先要考虑的是最常见的应用场景，最常见的应用场景就是deployment, 即无状态的副本集合；鱼与熊掌不可兼得，kubernetes对短任务调度的支持我认为是不如mesos来的直接。

![k8s-mesos](/relation.png)

Mesos是为解决特定问题所生，只是能够用来解决通用问题，比如Marathon, 一个使用mesos做调度的容器编排框架；kubernetes是为了解决通用问题而生，所以在特定问题上会显出冗余和掣肘。
Mesos从一开始就应该持有拥抱kubernetes的姿态，而不是竞争，mesos应该去弥补kubernetes在短任务调度中的不足。 我觉得在FaaS上，能看出kubernetes这种通用设计的局限性，并且mesos有它的优势。 faas倡导的是按需部署、按需计费，但是我们看基于kubernetes的faas解决方案（比如Fission、OpenFaas)是怎么实现的呢：把每种函数的负载（pod）用一个deployment(或者类似的自定义resources)关联起来，然后利用嵌入autoscaler机制，动态调整副本数。有两个问题很难解决：


1. 扩缩容策略。 我们也做过，但是复杂度可以说极高了。
2. 调度流程延迟对冷启动速度的影响。 scaler决策 -> pod创建/删除 -> k8s scheduler调度pod ->  kubelet启动pod。
3. 资源利用率的问题。函数的流量是波动，那么deployment的副本值是一段时间内函数的最高调用量决定的。当多个函数混部时，这就造成了“资源空洞” 。


如果我们用mesos去做的呢：
1. 扩缩容策略：不需要特别的扩容策略，有资源(offer)有任务就扩容； 缩容策略可以非常简单，定期删除idle函数实例或者主动删除idle函数实例。 
2. 调度流程简单： 创建删除pod ->  mesos-master -> mesos-slave。 
3. 资源利用率： 因为是真正的按需分配，所以就没有资源空洞，这正是mesos擅长的。

![k8s-mesos](/sharing.png)
