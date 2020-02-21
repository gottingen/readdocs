# Snapshots and Disk Utilization

使用nodetool snapshot命令创建快照时，Scyla不会像预期的那样将现有的SStable复制到快照目录。相反，它将创建到它们的硬链接。尽管这似乎微
不足道，但应注意以下几点：
* 快照磁盘空间最初将从零开始，并且将增长到与创建快照时节点数据集的大小相等的大小。因此，一开始磁盘空间利用率不会有任何显着增加。
* 尽管认为快照映像已立即创建似乎是合理的，但快照最终会增长到其预期大小
* 后台压缩将照常从SSTables中删除数据，但是任何具有硬链接（快照的一部分）的表都不会被删除。这将占用磁盘空间。
* 特别是，这意味着创建快照后，其“真实大小”从零开始，最终将等于创建快照时节点数据集的大小。

在此示例中，创建了快照。如您所见，其True size值为0个字节：

    
    nodetool listsnapshots
    Snapshot Details:
    
    Snapshot name Keyspace name Column family name True size Size on disk
    1574708464997 ks3           standard1          0 bytes   2.23 GB
    
    Total TrueDiskSpaceUsed: 0 bytes
一段时间之后，您可以看到其True大小与其在磁盘上使用的空间相同：

    
    nodetool compact ks3
    nodetool listsnapshots
    
    Snapshot Details:
    Snapshot name Keyspace name Column family name True size Size on disk
    1574708464997 ks3           standard1          2.23 GB   2.23 GB
    
    Total TrueDiskSpaceUsed: 2.23 GiB

    NOTE
    
    大型压缩使真正的快照大小在删除旧的SSTables时立即跳到数据大小。
    
## Additional References