# configure

系统配置步骤由Scylla RPM和deb软件包自动执行。

所有Scylla AMI和Docker映像均通过脚本按照以下步骤进行预配置。本文档仅供参考。

## System Configuration Files and Scripts


应应用几个系统配置设置。为了易于使用，提供了必要的脚本和配置文件。文件位于Scylla源代码中的dist / common和seastar / scripts下，并安装
在适当的系统位置。 （有关Scylla自己的配置文件的信息，请参见Scylla配置。）

### System Configuration Files

Source file	|Installed location	|Description
:---|:---|:---
limits.d/scylla.conf	|/etc/security/limits.d/scylla.conf	|Remove system resource limits
sysconfig/scylla-server	|/etc/sysconfig/scylla-server	|Server startup options
(written by scripts/scylla_coredump_setup , below)	|/etc/sysconfig/sysctl.d/99-scylla.conf	|Configure core dumps to use the scylla_save_coredump script

### Setup Scripts

记录了以下脚本，以供参考。它们全部由scylla_setup脚本调用，该脚本应在安装时或系统硬件更改时运行。

Source file	|Installed location	|Description
:---|:---|:---
scripts/scylla_bootparam_setup	|/usr/lib/scylla/scylla_bootparam_setup	|Set kernel options in bootloader
scripts/scylla_coredump_setup	|/usr/lib/scylla/scylla_coredump_setup	|Remove crash reporting software and set pattern for core dump names
scripts/scylla_ntp_setup	|/usr/lib/scylla/scylla_ntp_setup	|Configure Network Time Protocol
scripts/scylla_prepare	|/usr/lib/scylla/scylla_prepare	|Set up RAID and invoke network configuration
scripts/scylla_raid_setup	|/usr/lib/scylla/scylla_raid_setup	|Configure RAID and make XFS filesystem
scripts/scylla_run	|/usr/lib/scylla/scylla_run	|Wrapper to run Scylla with arguments from environment
scripts/scylla_save_coredump	|/usr/lib/scylla/save_coredump	|Compress a core dump file (Ubuntu only)
scripts/scylla_stop	|/usr/lib/scylla/scylla_stop	|Reset network mode if running in virtio or DPDK mode
scripts/scylla_sysconfig_setup	|/usr/lib/scylla/sysconfig_setup	|Rewrite the /etc/sysconfig/scylla file
seastar/scripts/posix_net_conf.sh	|/usr/lib/scylla/posix_net_conf.sh	|Set up networking options

### Bootloader Settings

如果Scylla安装在Amazon AMI上，则引导加载程序应提供clocksource = tsc和tsc = reliable选项。这样就可以使用精确的高分辨率时间戳计数器
（TSC）来设置系统时间。

此配置在文件/ usr / lib / scylla / scylla_bootparam_setup中提供

### Remove Crash Reporting Software

删除apport-noui或abrt软件包（如果存在），并为核心转储设置位置和文件名模式。

此配置在文件/ usr / lib / scylla / scylla_bootparam_setup中提供。

### Set Up Network Time Synchronization

强烈建议在Scylla服务器之间实施时间同步。

在所有节点上运行ntpstat以检查系统时间是否已同步。如果您在虚拟环境中运行，并且在主机上设置了系统时间，则可能不需要在客户机上运行NTP。查看
平台的文档。

如果使用Scylla与应用程序共享自己的时间服务器，请使用与应用程序服务器相同的NTP配置。脚本/ usr / lib / scylla / scylla_ntp_setup提供
合理的默认值，如果安装在Amazon云上则使用Amazon NTP服务器，否则使用其他池NTP服务器。

### Set Up RAID and Filesystem

对于生产而言，将文件系统设置为XFS是最重要和强制的。如果没有Scylla，Scyla的速度将大大降低。

脚本/usr/lib/scylla/scylla_raid_setup为Scylla执行必要的RAID配置和XFS文件系统创建。

该脚本的参数是

* -d specify disks for RAID
* -r MD device name for RAID
* -u update /etc/fstab for RAID

在Scylla AMI上，RAID配置会在

/usr/lib/scylla/scylla_prepare script

### CPU Pining

在安装Scylla时，强烈建议使用scylla_setup脚本。 Scylla不应与任何占用CPU的进程共享CPU。此外，在AWS上，由于相同的原因，我们建议将所有
NIC IRQ固定到CPU0。因此，应避免Scylla在CPU0及其超线程同级上运行。要验证Scylla是否固定CPU0，请使用以下命令：如果节点上的CPU少于四个，
则不要使用此选项。

核实
    
    cat /etc/scylla.d/cpuset.conf
    

输出示例：

    --cpuset `1-15,17-31`
    
### Networking


On AWS

1。防止irqbalance改变您的NIC的IRQ。
2. 将所有网卡的硬件队列绑定到CPU0：

    
    for irq in `cat /proc/interrupts | grep <networking iface name> | cut -d":" -f1`
    do echo "Binding IRQ $irq to CPU0" echo 1 > /proc/irq/$irq/smp_affinity done

3. 启用RPS并将RPS队列绑定到除CPU0及其超线程同级之外的CPU。
4. 启用XPS，并在所有可用CPU之间分配所有XPS队列。


posix_net_conf.sh脚本完成上述所有操作。

### On Bare Metal Setups with Multi-Queue NICs

1。防止irqbalance改变您的NIC的IRQ。
2. 将每个NIC的IRQ绑定到单独的CPU。
3. 启用XPS的方式与上述AWS完全相同。
4. 为listen（）套接字积压和未确认的挂起连接积压设置更高的值：

       
     echo 4096 > /proc/sys/net/core/somaxconn
     echo 4096 > /proc/sys/net/ipv4/tcp_max_syn_backlog
带有-mq参数的posix_net_conf.sh脚本可以完成上述所有操作。

## Configuring Scylla

see [scylla configure](/scylla/docs/ops/scylla-confiugure.md)

## Development System Configuration


在为Scylla提供DPDK支持时，请启用大页面。

    
    NR_HUGEPAGES=128
    mount -t hugetlbfs -o pagesize=2097152 none /mnt/huge
    mount -t hugetlbfs -o pagesize=2097152 none /dev/hugepages/
    for n in /sys/devices/system/node/node?; do
        echo $NR_HUGEPAGES > $n/hugepages/hugepages-2048kB/nr_hugepages;
    done

大页配置被 /usr/lib/scylla/sysconfig_setup写在 /etc/sysconfig/scylla-server

## Related Topics

[System Limits](/scylla/docs/design/system-limited.md) - outlines the system limits which should be set or removed