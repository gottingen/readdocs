# Scylla consistency quiz for administrators

:  当运行 nodetool decommission 删除一个节点时：
1. 节点拥有的令牌范围是否将重定位到其他节点？ vnodes呢？
2. 我们何时以及如何知道“退役”操作已完成？

A1: 是。节点在将其数据流传输到其他节点时进入状态“ STATE_LEAVING”。当数据流化后，它就在“ STATE_LEFT”中。

A2: 使用nodetool netstats检查节点状态

Q： 假设我有一个16节点集群，它在2个数据中心中使用网络拓扑策略。每个数据中心的复制因子均为2（DC1：2，DC2：2）。如果使用LOCAL_QUORUM进行
写入，则会将数据写入4个节点（每个数据中心2个），但是何时会发生确认？

A: quorum对于数据中心是本地的，因此将有2个本地写入和2个本地读取。异步地，本地协调器会将写操作发送到远程协调器，但是本地读和写操作不需要它