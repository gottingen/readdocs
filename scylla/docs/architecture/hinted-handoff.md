# Scylla Hinted Handoff



Scylla中的典型写法是根据我们的[容错文档](/scylla/docs/architecture/fault-tolerance.md)中描述的方案进行的。

但是，将写请求发送到Scylla节点却由于诸如节点上的大量写负载，网络问题甚至硬件故障等原因而无响应时，会发生什么？为了确保可用性和一致性，
Scyla实现了自动切换。


提示=目标副本ID +突变数据

换句话说，Scylla保存用于停机节点的写入副本，并在以后启动时将其重播到节点。因此，当节点关闭时，写操作流程如下所示：

1. 协调器确定所有副本节点；
2. 协调器根据复制因子（RF）尝试写入RF节点。

![1-write](/scylla/images/1-write_op_RF_31.jpg)

3. 如果一个节点关闭，则仅从两个节点返回确认：

![hinted-handoff-3](/scylla/images/hinted-handoff-3.png)

4. 如果一致性级别不需要所有副本的响应，则在这种情况下，协调器V将响应客户端写入成功的信息。协调器将为丢失的节点编写并存储提示：

![hint-handoff-4](/scylla/images/hinted-handoff-4.png)

5. 一旦出现故障节点，协调器将重播该节点的提示。协调器收到写入确认后，将删除提示。

![hind-handoff-5](/scylla/images/hinted-handoff-5.png)

协调器在以下情况下存储切换提示：

1. 宕机节点
2. 超时时间内未收到复制节点的恢复 write_request_timeout_in_ms

如果节点的停机时间超过了该节点的时间，协调器将停止为该节点创建任何提示 max_hint_window_in_ms