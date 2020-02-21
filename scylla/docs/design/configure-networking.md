# Configure Scylla Networking with Multiple NIC/IP Combinations 

在scylla.yaml中，有许多方法可以配置IP地址。 IP地址设置不正确，可能会导致效果不理想。本文重点介绍对网络通信至关重要的地址。

本文包含在scylla.yaml中配置网络的不同方法的示例。地址配置的整个范围在管理指南中。

由于这些值取决于您设置中的特定网络配置，因此有几种方法可以配置上述这四个地址参数。在下面的示例中，我们将提供最常见用例的说明（全部以单个Scylla节点的分辨率表示）。

## 1 NIC, 1 IP

在这种情况下，Scylla群集应在具有单个地址空间（没有“公共/内部IP”）的单个子网中运行。

在这种情况下：

* IP = node’s IP address

        listen_address: IP
        rpc_address: IP
        broadcast_address: not set
        broadcast_rpc_address: not set
        endpoint_snitch: no restrictions

## 1 NIC, 2 IPs
使用公共和内部IP地址时就是这种情况。具有公共IP地址的AWS实例是此配置的经典示例。

在这种情况下：

* IPp = the node’s public IP address
* IPi = the node’s internal IP address

IPi是配置为与NIC相同的IP的IP（从OS角度来看），并且在运行命令ifconfig <iface>时，输出将显示此IP地址。

    listen_address: IPi
    rpc_address: IPi
    broadcast_address: IPp
    broadcast_rpc_address: IPp
    endpoint_snitch: GossipintPropertyFileSnitch with prefer_local=true | any of Ec2xxxSnitch snitches.

## 2 NICs, 2 IPs

在这种情况下，用户希望CQL请求或响应通过一个子网（net1）发送，并且节点间通信通过另一个子网（net2）进行。

在这种情况下：

* IP1 = node’s IP in net1 subnet
* IP2 = node’s IP in net2 subnet

        listen_address: IP2
        rpc_address: IP1
        broadcast_address: not set
        broadcast_rpc_address: not set
        endpoint_snitch: no restrictions
