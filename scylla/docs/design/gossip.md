# Gossip in Scylla

与Apache Cassandra一样，Scylla使用一种称为“ gossip”的协议来交换有关集群中节点身份以及节点处于运行状态还是处于关闭状态的元数据。当然，
由于没有单点故障，所以节点状态也就没有单一的注册表，因此节点之间必须共享信息。

gossip协议仅在分布式系统中才需要，因此对于大多数管理员来说可能是新的。根据[Wiki](https://en.wikipedia.org/wiki/Gossip_protocol)的
说法，理想的gossip协议具有以下几种品质：

* gossip涉及节点之间的周期性成对交互
* 节点之间交换的信息具有限制大小。
* 至少一个代理的状态更改以反映另一个代理的状态。
* 不能进行可靠的通信。
* 与典型的消息等待时间相比，交互的频率较低，因此协议成本可忽略不计。
* 对等选择中存在某种形式的随机性。
* 由于复制，所传递的信息存在隐式冗余。

像Apache Cassandra一样，Scylla中的个人gossip互动相对少见且简单。每个节点每秒一次随机选择1到3个节点进行交互。一个可以是任何活动节点，一
个可以是种子节点，并且可以从标记为不可访问的节点中选择一个。

种子节点最有可能被选为gossip 互动节点，因为有三个随机选择。种子节点有机会进入第一组，然后（如果不是第一次选择种子）则进入第三组。

每个节点每秒运行一次gossip协议，但是gossip运行不会在集群中同步。

## One round = three messages

一轮gossip 互动由三则讯息组成。 （我们将称为启动圆形节点A的节点，以及随机选择的节点Node B）。

* Node A sends: gossip_digest_syn
* Node B sends: gossip_digest_ack
* Node A replies: gossip_digest_ack2

虽然名称是从TCP借用的，但闲话不需要在节点之间建立新的TCP连接。

## What are nodes gossiping about?

节点之间交换少量信息。主要的两个数据结构是heart_beat_state和application_state。


heart_beat_state包含用于生成的整数和“版本号”。生成是每次启动节点时都会增长的数字，而版本号是覆盖应用程序状态版本的不断增加的整数。 
ApplicationState包含有关节点内组件状态（例如负载）的数​​据和版本号。每个节点为群集中的所有节点（包括其自身）维护节点IP地址和节点 gossip 
元数据的映射。

一轮gossip旨在最大限度地减少发送的数据量，同时解决两个gossip节点上的节点状态数据之间的任何冲突。在gossip_digest_syn消息中，节点A发送一个闲话
摘要：所有已知节点，世代和版本的列表。节点B将生成和版本与其已知节点进行比较，并在gossip_digest_ack消息中发送其自身不同的任何数据以及其自
身的摘要。最后，节点A会回复其已知状态与节点B的摘要之间的状态差异。

## Scylla gossip implementation


Scylla gossip消息以及所有其他节点间流量（包括发送变异和数据流）都在Scylla messages_service上运行。 Scylla的messages_service在Seastar 
RPC服务上运行。 Seastar是Scylla使用的多核系统的可扩展软件框架。如果一对节点之间没有建立TCP连接，则messages_service将创建一个新的节点。
如果已经启动，则消息传递服务将使用现有的消息传递服务。

## Gossip on multicore

每个Scylla节点均由几个独立的分片组成，每个内核一个，它们在不共享的基础上运行，并且通信时不会锁定。在内部，仅在CPU 0上运行的gossip组件需要具
有从其他分片转发的连接。gossip共享的节点状态数据将复制到其他分片。

gossip协议提供了重要的优势，尤其是对于大型集群。与跨节点的“泛洪”信息相比，它可以更快地同步数据，并允许在新节点关闭或节点恢复服务时快速恢复。
节点仅在检测到实际故障时才将其他节点标记为已关闭，但gossip很快就分享了一个节点恢复正常的好消息。

## References

[Cassandra Wiki: ArchitectureGossip](http://wiki.apache.org/cassandra/ArchitectureGossip)

[Apple Inc.: Cassandra Internals — Understanding Gossip](https://www.youtube.com/watch?v=FuP1Fvrv6ZQ&list=PLqcm6qE9lgKJkxYZUOIykswDndrOItnn2&index=49)

[Using Gossip Protocols For Failure Detection, Monitoring, Messaging And Other Good Things, by Todd Hoff](http://highscalability.com/blog/2011/11/14/using-gossip-protocols-for-failure-detection-monitoring-mess.html)

[Gossip protocol on Wikipedia](https://en.wikipedia.org/wiki/Gossip_protocol)
