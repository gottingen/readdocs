# glossary

* anti-entropy

    
    数据有序、有组织的状态，Scylla拥有适当的流程来确保数据是反熵的，其中所有副本都包含最新数据，并且副本之间的数据保持一致。
    [anti-entropy](/scylla/docs/topics/anti-entropy.md)
    
* bootstrap


    将新节点添加到群集时，引导过程将确保将群集中的数据自动重新分发到新节点。在这种情况下，新节点是没有系统表或数据的空节点。
    [bootstrap]()
    
* cap theorem
    
    
    CAP定理是在分布式系统中，数据的C（一致性），A（可用性）和P（分区容差）相互依赖的概念。增加其中任何两个因素将减少第三个因素。 
    Scylla选择可用性和分区容忍度而不是一致性。
    
* cluster

    
    一个或多个Scylla节点协同工作，它们拥有单个连续的令牌范围。状态通过Gossip协议在群集中的节点之间进行通信.
    
* clustering key


    单列或多列集群键确定分区中磁盘上行的唯一性和排序顺序
    
* column family


    
* compaction


    读取多个SSTable，比较数据和时间戳，然后写入一个包含合并的最新信息的SSTable的过程。
    
* compaction strategy


    确定将压缩哪个SSTable，以及何时压缩
    
* consistency level (CL)


    一个动态值，该值指示必须确认读或写操作的副本数（在群集中）。客户端根据每个操作设置此值。
    
* consistency level: All
    
    
    必须将写入操作写入集群中的所有副本，而读取则等待来自所有副本的响应。提供最低的可用性和最高的一致性
    
* consistency level: Any


    必须将写入写入集群中的至少一个副本。读取等待至少一个副本的响应。以最低的一致性提供最高的可用性
    
* Consistency Level: Each_quorum
    
    
    仅支持必须写入所有数据中心中的副本副本仲裁的写入。
    
* Consistency Level: Local_one
    
    
    本地数据中心中至少有一个副本响应
    
* Consistency Level: Local_quorum


    本地数据中心中的法定副本数作出响应。
    
* Consistency Level: One


    群集中只有一个副本需要响应。
    
* Consistency Level: Quorum

    
    Quorum是跨整个群集（包括所有数据中心）的全局一致性级别设置。使用Quorum作为一致性级别时，协调器必须等待大多数节点确认后，才能兑现
    请求。如果RF = 3，那么至少2个副本必须响应。可以使用公式（n / 2 +1）计算QUORUM，其中n是复制因子。如果您有两个数据中心，则两个数据
    中心中的所有节点均计入法定多数。例如，有一个群集，其中有两个DC，一个DC中有三个节点，另一个DC中有两个节点。如果较小的DC失败，则请求
    仍将在Quorum下传递为3> 5/2。
    
* Date-tiered compaction strategy (DTCS)
    
    
    DTCS专为时间序列数据而设计，但不应使用。
    
* Entropy


    数据不一致的状态。当副本不同步并且数据是随机的时，这就是结果。 Scylla采取了预防性措施。
    
* Eventual Consistency
    
    
    在Scylla中，当考虑CAP定理时，可用性和分区容忍度被认为是比一致性更高的优先级。
    
* Hint
    
    
    协调器保留的写请求的简短记录，直到无响应的节点再次变得响应为止，此时提示中的写请求数据将被写入副本节点。
    
* Hinted Handoff
    
    
    减少当节点关闭或网络拥塞时可能发生的数据不一致。在Scylla中，当数据被写入并且有无响应的副本时，协调器会向自己写入提示。当节点恢复时，
    协调器会向该节点发送待处理的提示，以确保该节点具有应接收的数据。
    
* JBOD


    BOD或“只是另一堆磁盘”是一种非raid的存储系统，它使用具有多个磁盘的服务器来实例化每个磁盘的单独文件系统。好处是，如果单个磁盘发生故
    障，则只需要更换它，而不是整个磁盘阵列。缺点是自由空间和负载可能分布不均。
    
* Key Management Interoperability Protocol (KMIP)


    KMIP是一种通信协议，定义用于在密钥管理服务器（KMIP服务器）上存储密钥的消息格式。使用静态加密时，可以使用KMIP服务器保护密钥。
    
* Keyspace

    
    具有属性的表的集合，这些属性定义了如何在节点上复制数据。
    
* Leveled compaction strategy (LCS)
    
    
    LCS使用固定大小的小型SSTable（默认为160 MB），分为不同级别。

* Log-structured-merge (LSM)


    保留排序的文件并合并它们的技术。 LSM是维护键值对的数据结构。
    
* Logical Core (lcore)
    
    
    超线程系统上的超线程核心，或没有超线程的系统上的物理核心。
    
* MemTable

    
    内存中的数据结构既为读取又为写入提供服务。一旦填满，Memtable将刷新到SSTable。
* Mutation
    
    
    对数据的更改（例如要插入的一个或多个列）或删除。
    
* Node
    
    
    Scylla的一个已安装实例。
    
* Nodetool
    
    
    一个简单的命令行界面或管理Scylla节点。 nodetool命令可以显示给定节点的公开操作和属性。 Scylla的nodetool包含这些操作的一部分。
    
* Partition
    
    
    存储在节点上并在节点之间复制的数据子集。有两种考虑分区的方法。在CQL中，分区显示为一组排序的行，并且是查询数据的访问单位，因为大多数
    查询都访问单个分区。在物理层上，分区是存储在节点上的数据单元，并由分区键标识。
* Partition Key


    分区的唯一标识符，即分区键，可以从主键的第一列中散列出来。分区键也可以从一组列中散列，通常称为复合主键。分区键确定哪个虚拟节点获取第
    一个分区副本。
    
* Partitioner


    散列函数，用于计算哪些数据存储在集群中的哪个节点上。分区程序将分区键作为输入，并返回环形令牌作为输出。默认情况下，Scylla使用64位
    Murmurhash3函数，并且此哈希范围以数字形式表示为无符号64位整数，
    
* Primary Key
    
    
    在CQL表定义中，主键子句指定分区键和可选的群集键。这些键唯一地标识分区中的每个分区和行。
    
* Quorum (Consistency Level)

* Read Amplification
    
    
    
    过多的读取请求需要许多SSTables。 RA是通过每个查询读取的磁盘数来计算的。当要阅读许多页面以回答查询时，会出现高RA。

* Read Operation


    当应用程序从SSTable获取信息并且不以任何方式更改该信息时，就会发生读取操作。

* Read Repair
    
    
    一种用于读取操作的反熵机制，可确保使用最新更新的数据来更新副本。这些修复会在后台自动，异步运行。
    
* Reconciliation
    
    
    数据迁移期间的验证阶段，将目标数据与原始源数据进行比较，以确保迁移体系结构正确传输了数据。
    
* Repair
    
    
    一个在后台运行并在节点之间同步数据的进程，以便最终所有副本都拥有相同的数据。
    
* Replication
    
    
    在群集中的各个节点之间复制数据的过程。
    
* Replication Factor
    
    
    给定集群中副本节点的总数。 RF为1表示数据将仅存在于群集中的单个节点上，并且没有任何容错能力。此数字是为每个键空间定义的设置。所有副
    本具有相同的优先级；没有主副本或主副本。可以为每个DC定义一个RF。
    
* Replication Factor (RF)
    
    
    给定集群中副本节点的总数。 RF为1表示数据将仅存在于群集中的单个节点上，并且没有任何容错能力。此数字是为每个键空间定义的设置。所有副
    本具有相同的优先级；没有主副本或主副本。可以为每个DC定义一个RF。
        
* Size-tiered compaction strategy
    
    
       当系统具有足够数量（默认为四个）大小相似的SSTables时触发。
       
* Snapshot
    
    
    Scylla中的快照是备份和还原机制的重要组成部分。在其他数据库中，备份是从创建数据文件的副本（冷备份，热备份，卷影副本备份）开始的，
    而在Scylla中，备份过程是从创建表或键空间快照开始的。
    
* Space amplification
    
    
    磁盘空间使用过多，要求磁盘大于完全压缩的数据表示形式（即，单个SSTable中的所有数据）。 SA计算为磁盘上数据库文件大小与实际数据大小之
    比。当使用的磁盘空间多于数据大小时，将发生高SA。
    
* SSTable


    从Google Big Table，SSTables或Sorted String Tables借用的概念存储了一系列不可变的行，其中每行均由其行键标识。请参阅压缩策略。
     SSTable格式是一种持久性文件格式。
    
* Table


    行收集的列的集合。列按聚类键排序。
    
* Time-window compaction strategy
    
    
    TWCS专为时间序列数据而设计，并替代了日期分层压缩。
    
* Token
    
    
    有范围的值，用于标识节点和分区。 Scylla群集中的每个节点都被赋予一个（初始）令牌，该令牌定义了节点处理范围的末端。
    
* Token Range
   
    
    分区程序支持的潜在唯一标识符的总范围。默认情况下，群集中的每个Scylla节点处理256个令牌范围。每个令牌范围对应于一个Vnode。每个哈希范
    围又是给定哈希函数总范围的一部分。
   
* Tunable Consistency
    
    
    每个查询的唯一性，一致性级别设置的可能性。这些是增量的，并且会覆盖旨在增强数据一致性的固定数据库设置。当给定查询或操作的响应速度更为
    重要时，可以直接从CQL语句设置此类设置。
    
* Virtual node
    
    
    单个Scylla节点拥有的令牌范围。 Scylla节点是可配置的，并支持一组Vnode。在传统令牌选择中，一个节点每个节点拥有一个令牌（或令牌范围
    ）。使用Vnode，一个节点可以拥有许多令牌或令牌范围。在集群中，可以从非连续的集合中随机选择它们。在Vnode配置中，每个令牌都位于特定的
    令牌范围内，该令牌范围又表示为Vnode。然后，将每个Vnode分配给群集中的物理节点。
    
* Write Amplification
    
    
    过度压缩相同的数据。 WA由写入存储的字节数与写入数据库的字节数之比计算得出。当写入存储的字节/秒多于实际写入数据库的字节/秒时，
    发生高WA。
* Write Operation

    
    从SSTable添加信息或从SSTable删除信息时，都会发生写操作。
    

