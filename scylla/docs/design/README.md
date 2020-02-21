# scylla design

* [gossip](/scylla/docs/design/gossip.md)
* [compaction](/scylla/docs/design/compaction.md)
* [io](/scylla/docs/design/io.md)

## Planning and Setup

* [system limited](/scylla/docs/design/system-limited.md) - 概述应设置或删除的系统限制
* [DPDK mode](/scylla/docs/design/dkdp.md) - 了解为DPDK模式选择和配置网络
* [Compaction](/scylla/docs/design/compaction.md) - 为了释放磁盘空间并加快读取速度，Scyla必须执行压缩操作。
* [Operating System (OS) Support Guide](/scylla/docs/design/system-support.md) - 操作系统支持。
* [POSIX networking for Scylla](/scylla/docs/design/posix-net.md) - Scylla的POSIX模式可在所有物理和虚拟网络设备上使用，对开发工
作很有用。

## Scylla under the hood

* [Gossip in Scylla](/scylla/docs/design/gossip.md) - 与Cassandra一样，Scylla使用一种称为“gossip”的协议来交换有关集群中节点身份
的元数据。幕后工作原理如下。

## Configuring and Integrating Scylla