# POSIX networking for Scylla

Scylla中使用的Seastar框架可以支持两种联网模式。对于高性能的生产工作负载，请使用数据平面开发套件（DPDK）在特定的现代网络硬件上获得最佳性
能。

为了便于开发，您还可以使用POSIX模式，该模式适用于操作系统支持的所有物理和虚拟网络设备。

## Kernel Configuration

在POSIX模式下，建议（但不是必需）Linux“transparent hugepages”功能，以最大程度地减少内存分配的开销。
 /sys/kernel/mm/transparent_hugepage/enabled 的值应为always 或者 madvise。有关如何启用 transparent hugepages的说明，
 请参见Linux发行版的文档。
 
 ## Firewall Configuration
 对于单个节点，将需要设置防火墙以允许 scylla 使用的 TCP ports
 
 ## Scylla Configuration
 
 默认是用posix模式， 在/etc/sysconfig/scylla-server文件中。确认NETWORK_MODE设置为posix
 
    # choose following mode: virtio, dpdk, posix
    NETWORK_MODE=posix.
    
在 POSIX 模式下，选项ETHDRV将被忽略。

## More Information

* [Cassandra: configuring firewall port access](http://docs.datastax.com/en//cassandra/2.0/cassandra/security/secureFireWall_r.html)
* [How to use, monitor, and disable transparent hugepages in Red Hat Enterprise Linux 6](https://access.redhat.com/solutions/46111) 

 