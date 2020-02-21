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

* [Scylla consistency quiz for administrators](/scylla/docs/design/consistency-quiz.md) - 从管理员的角度，您对NoSQL有多少了解？

* [Data model for a social reader application ](/scylla/docs/design/data-model.md) - 链接共享和推荐站点的简单数据模型

* [Scylla Memory Usage ](/scylla/docs/design/memory-usage.md) - 简短说明Scylla如何管理内存

* [Scylla Nodes are Unresponsive ](/scylla/docs/design/swap.md) - 如何处理Scylla中的交换

* [CQL Query Does Not Display Entire Result Set ](/scylla/docs/design/cql-no-display.md) - 当CQL查询未显示整个结果集时该怎么办。

* [Snapshots and Disk Utilization](/scylla/docs/design/disk-utilization.md) - 快照如何影响磁盘利用率

* [Scylla Snapshots](/scylla/docs/design/snapshot.md) - Scylla快照是什么，它们的用途以及如何创建和删除它们。
## Configuring and Integrating Scylla

* [NTP configuration for Scylla](/scylla/docs/design/ntp-configure.md) - Scylla取决于准确的系统时钟。了解为数据存储和应用程序配置NTP。
* [Scylla and Spark integration](/scylla/docs/design/spark-in.md) - 如何运行使用Scylla存储数据的示例Spark应用程序？
* [Map CPUs to Scylla Shards ](/scylla/docs/design/map-cpu.md) - CPU和Scylla分片之间的映射重新创建RAID设备-如何在不运行scylla-setup的情况下重新创建RAID设备
* [Configure Scylla Networking with Multiple NIC/IP Combinations](/scylla/docs/design/configure-networking.md) - 在scylla.yaml中设置不同IP地址的示例
* [Kafka Sink Connector Quickstart](/scylla/docs/design/kafka-quick.md)
* [Kafka Sink Connector Configuration](/scylla/docs/design/kafka-configure.md)

## Analyzing Scylla

* [Using the perf utility with Scylla](/scylla/docs/design/perf.md) - 使用perf分析Scylla
* [Debug your database with Flame Graphs](/scylla/docs/design/flame.md) - 如何设置和运行火焰图
* [Decoding Stack Traces](/scylla/docs/design/stack.md) - 如何在Scylla日志中解码堆栈跟踪