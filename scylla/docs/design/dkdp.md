# dpdk

Scylla设计为使用Seastar框架，该框架使用Data Plane Development Kit（DPDK）直接驱动NIC硬件，而不是依赖内核的网络堆栈。这为Scylla提供
了巨大的性能提升。 Scylla和DPDK还依靠Linux的“大页面”功能来最大程度地减少内存分配的开销。各种高性能网络设备都支持DPDK。

Brand	|Device	|Status
:--- |:---|:---
Intel	|ixgbe (82598..82599, X540, X550)	| yes
Intel	|i40e (X710, XL710)	| yes

Scylla RPM软件包带有DPDK支持，但是该软件包默认为POSIX网络模式（请参阅管理指南）。要启用DPDK，请编辑/etc/sysconfig/scylla-server并编
辑以下几行：

    
    # choose following mode: virtio, dpdk, posix
    NETWORK_MODE=posix
    
    # Ethernet device driver (dpdk)
    ETHDRV=

## Reference

[DPDK: Supported NICs](http://dpdk.org/doc/nics)

[design base](/scylla/docs/design/README.md)

