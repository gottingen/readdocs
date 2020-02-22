# install scylla on cnetos7 

## Prerequisites

* CentOS 7.3 or later, for the 64-bit x86_64 architecture.
* Yum package management application installed.
* ABRT conflict with Scylla coredump configuration. Remove it before installing Scylla: sudo yum remove -y abrt
* Root or sudo access to the system.


确保所有相关 port 均已打开。

## Procedure


    sudo yum install epel-release

## Configure and run Scylla on CentOS

Configure Scylla

Configure the /etc/scylla/scylla.yaml file with the following parameters:

Item	| Content
:--- | :---
cluster_name	|Name of the cluster, all the nodes in the cluster must have the same name
seeds	|Seed nodes are used during startup to bootstrap the gossip process and join the cluster
listen_address	|IP address that the Scylla use to connect to other Scylla nodes in the cluster
rpc_address	|IP address of interface for client connections (Thrift, CQL)

### Scylla setup

Run the scylla_setup script to tune the system settings

    
    sudo scylla_setup
    
该脚本调用一组脚本来配置多个操作系统设置，例如设置RAID0和XFS文件系统。它还会在您的存储上运行短暂（最多几分钟）的基准测试，并生成
/etc/scylla.d/io.conf配置文件。文件准备好后，您可以启动Scylla（请参见下文）。没有XFS或io.conf文件，Scylla将无法运行。要绕过此检查，
请将Scylla设置为开发人员模式。

Run Scylla as a service (if not already running)

    sudo systemctl start scylla-server
    
run nodetool


    nodetool status
    
run cqlsh

    
    cqlsh
    
Run cassandra-stress

    
    cassandra-stress write -mode cql3 native 
    
## Monitoring


强烈建议将Scylla监视堆栈安装到位。有关如何在此使用Grafana设置Scylla监控的更多信息


