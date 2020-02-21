# Scylla Nodes are Unresponsive

issue:

    Scylla节点无响应。它们显示为关闭，我什至无法建立与群集的新SSH连接。现有的连接速度很慢。
    
## Root Cause

当Scylla报告自身故障时，这可能意味着特定于Scylla的问题。但是，当整个节点开始报告运行缓慢，甚至很难建立SSH连接时，通常表明节点级别存在问题。

最常见的原因是由于交换。我们需要考虑两种主要情况：

* 系统已配置交换。如果系统需要交换页面，则可能会交换Scylla内存，并且将来对该内存的访问将很慢。
* 系统未配置交换。在那种情况下，内核可能会循环尝试释放页面而无法这样做，从而成为CPU占用者，最终使Scylla和其他进程停止执行。

## Resolution


理想情况下，健康的系统不应互换。默认情况下，Scylla会预分配93％的内存，并且永远不会使用更多的内存。它将剩余的7％内存留给其他任务
（包括操作系统）使用。请与top实用程序一起检查是否有其他正在运行的进程正在消耗大量内存。

* 如果有其他进程正在运行，但不是必需的，我们建议将其移至其他计算机。
* 如果有其他进程正在运行并且它们是必不可少的，则默认保留可能不够。请按照以下步骤更改预订。


启用交换而不使用它比需要交换而不使用它更好。配置文件或分区以用作生产部署的交换。

## Change memory reservation

Add --reserve-memory \[memory\] to the scylla command line at:

/etc/sysconfig/scylla-server (RHEL variants) or
/etc/defaults/scylla-server (Debian variants)
For example --reserve-memory 10G (will reserve 10G)

