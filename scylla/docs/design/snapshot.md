# Scylla Snapshots

## Synopsis

Scylla中的快照是备份和还原机制的重要组成部分。在其他数据库中，备份是从创建数据文件的副本（冷备份，热备份，卷影副本备份）开始的，而在
Scylla中，备份过程是从创建表或键空间快照开始的。快照是自动创建的（本文将对此进行详细描述），也可以通过调用nodetool snapshot命令来创建。
为防止数据还原出现任何问题，备份策略必须包括将快照的副本保存在辅助存储上。这样可以确保在主存储出现故障时可以恢复快照。

    
    NOTE
    
    如果您来自RDBMS背景，则不应将快照与实例化视图的概念混淆（因为它们在该技术领域有时被称为快照）。使用Scylla，快照是指向数据文件的硬链
    接。实例化视图确实存在于Scylla中，但不称为快照。

## How Snapshots Work

与Cassandra一样，Scylla也需要类似Unix的存储（这也是Linux支持的文件系统）。如上所述，快照是到磁盘上SSTables的硬链接。重要的是要了解SSTables是不可变的，因此不能在同一文件中重写。当数据库中的数据更改并将数据写入磁盘时，它将作为新文件写入。压缩后将合并新文件，压缩会将表的数据合并到一个或多个SSTable文件中（取决于压缩策略）。

如果在磁盘上创建了到现有SSTables的快照（硬链接），则即使最终将表数据存储在一个或多个新SSTables中，快照也会保留下来。压缩过程将删除数据目录中的文件，但是快照硬链接仍将指向旧文件。仅在删除所有指针之后，才删除实际文件。即使存在一个指针，文件也将保留。因此，即使数据库正在运行，一旦创建快照硬链接，就可以将数据文件的内容复制到另一个存储中，并用作表，键空间或整个数据库还原的基础（在该节点上） ，因为此备份和还原过程是特定于节点的）。

除了上述计划的备份过程之外，为了防止意外丢失数据，Scyla数据库还包括可选的每次删除或截断表时自动快照的创建。由于删除键空间涉及删除该键空间内的表，因此这些操作也将调用自动快照。此选项开箱即用，由/etc/scylla/scylla.yaml配置文件中的auto_snapshot标志控制。请注意，键空间不能被截断。它只能被丢弃。另一方面，表可以被截断或删除。表中的数据也可以删除，这与被截断不同。

/etc/scylla/scylla.yaml文件中auto_snapshot标志的默认设置为true。不建议将其设置为false，除非有适当的备份和恢复策略。

## Snapshot Creation

Snapshots are created when:

* nodetool snapshot <kespace_name>运行，因为这将为键空间中的所有表创建快照。

* nodetool snapshot <kespace_name>.<table_name> 在为特定表创建快照时运行
* auto_snapshot (在scylla.yaml中)设置为true 并且表被删除（DROP TABLE <kespace_name>.<table_name>）
* auto_snapshot (在scylla.yaml中)设置为true并且表被截断（TRUNCATE TABLE <kespace_name>.<table_name>）
* auto_snapshot (在scylla.yaml中)设置为true 并且keyspace被删除(DROP KEYSPACE <kespace_name>) 等价于所有keyspace中表。

Snapshots are not created when：
* 表中的数据被删除，而不是被截断
* auto_snapshot标志（在scylla.yaml中）设置为false，并且该表已删除或被截断。
  
##  List Current Snapshots

调用命令时数据库中存在的表或键空间的快照信息可以与nodetool listnapshots一起显示。即使快照仍然存在于磁盘上，也无法列出任何引用了已删除键空间或已删除表的快照（因为有关这些快照的数据库信息已删除，但快照仍保留在磁盘上）。您可以通过在Linux中使用find命令查找快照目录来找到这些快照。

## Remove Snapshots

快照不会自动删除。在活动的数据库节点上，旧快照占用磁盘空间，需要手动将其删除以释放存储空间。更改原始数据文件后，快照只会在磁盘上占用更多空间。

    CAUTION

    不要在文件系统级别手动删除快照，请使用nodetool命令。

使用此过程可删除快照或本地备份。

Procedure：

使用以下步骤之一：

要从特定的键空间中删除特定的快照：

* 运行noodetool clearsnapshot -t <快照名称> <kespace_name>。请注意，此命令中的快照名称是分配给该快照的标签，标签。

要从所有键空间中删除命名快照，也就是说，如果任何一个键空间碰巧都包含命名快照，请执行以下操作：

* 运行noodetool clearsnapshot -t <快照名称>命令。这里省略了键空间名称。

要删除所有现有快照而没有任何警告：

* 运行noodetool clearsnapshot

    
    CAUTION

    在不指定键空间或快照的情况下运行noodetool clearsnapshot时请格外小心，因为此命令不仅会删除“ nodetool listnapshots”命令列出的快照，还将删除节点存储上的所有其他快照，包括先前删除的表或键空间的快照。

当所有其他方法均失败时，您需要手动删除快照：

如果无法启动数据库，nodetool命令将无法列出或删除快照。如果在这种情况下必须清除存储中的旧快照，则剩下的唯一其他方法是使用Linux rm命令在存储级别手动删除快照。


    NOTE

    使用rm命令删除快照并返回无法启动的数据库后，nodetool listnapshots可能仍会列出手动删除的快照。

## Additional References



