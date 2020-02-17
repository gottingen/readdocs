# 总览

scylla 是一个可以横向、纵向扩展的数据库。scylla 采用Apache Cassandra项目的分布式设计（采用Amazon Dynamo的分布概念和Google 
BigTable的数据建模概念）。

在大数据的世界里，单节点不能承载整个数据集，因此需要一个节点集群。

一个scylla集群可视为一个节点或实例的集合的环。所有节都应该是同质的，使用无共享的方法。这篇文章描述了数据是如何在节点之间的分布决策。

Scylla keyspace 是具有属性的表集合，这些属性定义了如何在节点上复制数据的策略。

keyspace 类似于sql中的数据库。当一个新的 keyspace被创建，用户将设置一个数字属性，即复制因子，该属性定义了如何在节点上复制数据。例如，
RF为2意味着给定令牌或令牌范围将存储在2个节点上（或复制到一个其他节点上）。在示例中，我们将使用2的RF值。

表格是模式定义的列和行的标准集合。随后，在创建表时，使用键空间内的CQL（Cassandra Query Language），将在表列的子集中定义一个主键。

因此，下图中的表格可以在CQL中定义如下：
    
    CREATE TABLE users (
         ID int,
         NAME text,
         ADDRESS text,
         PHONE text,
         PHONE_2 text,
         PRIMARY KEY (ID)
    );

在CQL表定义中， primary key 子句至少指定一个单列分区键，并且还可以指定聚簇键列。 primary key唯一地标识表中的每个分区/行组合，而聚类
键指示在给定分区内如何对数据（行）进行排序。有关更多信息，请参见我们的CQL文档。

![table](/scylla/images/table-1.png)

token 是一个范围内的值，用于标识节点和分区。分区密钥( partition key )是分区的唯一标识符，并表示为从主密钥散列( primary key)的token。


分区是存储在节点上并在节点之间复制的数据子集。有两种考虑分区的方法。在CQL中，分区显示为一组排序的行，并且是查询数据的访问单位，因为大多
数查询都访问单个分区。在物理层上，分区是存储在节点上的数据单元，并由分区键标识。

在上图中，显示了3个分区，分区键为101、102和103。

分区键是查找包含分区的一组行的主要方法。分区键用于标识存储给定分区的集群中的节点，以及在集群中的各个节点之间分布数据。

分区程序或分区哈希函数使用分区键确定数据在群集中给定节点上的存储位置。它通过为每个分区键计算一个令牌来实现。默认情况下，使用Murmur3哈希
函数对分区键进行哈希处理。

![ring-table](/scylla/images/ring-architecture-2.png)

分区键的散列输出确定其在群集中的位置。

![ring-table-3](/scylla/images/ring-architecture-3.png)

上图显示了示例性的0-1200令牌范围，该范围在三节点群集中平均分配。

默认情况下，Scylla使用Murmur3分区程序。使用MurmurHash3函数，64位哈希值（为分区键生成）的范围为从-2^63到2^63-1。这解释了为什么下面的
nodetool ring 环输出中也存在负值。

![ring-table-4](/scylla/images/ring-architecture-4.png)

在上图中，每个数字代表一个令牌范围。复制因子为2时，我们看到每个节点距离上一个节点一个范围，而下一个节点一个范围。

但是请注意，Scylla仅使用面向Vnode的体系结构。虚拟节点表示单个Scylla节点拥有的令牌的连续范围。可以为物理节点分配多个非连续的Vnode。

Scylla的面向Vnode的体系结构实现具有多个优点。首先，添加或删除节点时不再需要重新平衡群集。其次，由于重建可以从所有可用节点（而不只是数
据驻留在每个节点一令牌的节点上）流式传输数据，因此Scylla可以更快地重建。

![ring-table-5](/scylla/images/ring-architecture-5.png)

可以在scylla.yaml的num_tokens设置中配置分配给群集中每个节点的Vnode的比例。默认值为256。

您可以使用nodetool命令来描述节点的不同方面以及它们存储的令牌范围。例如，

    $ nodetool ring <keyspace>

输出节点的所有令牌，并显示令牌环信息。它为单个数据中心产生如下输出：

    Datacenter: datacenter1
    =======================
    Address     Rack        Status State   Load            Owns                Token
                                                                             9156964624790153490
    172.17.0.2  rack1       Up     Normal  110.52 KB       66.28%            -9162506483786753398
    172.17.0.3  rack1       Up     Normal  127.32 KB       66.69%            -9154241136797732852
    172.17.0.4  rack1       Up     Normal  118.32 KB       67.04%            -9144708790311363712
    172.17.0.4  rack1       Up     Normal  118.32 KB       67.04%            -9132191441817644689
    172.17.0.3  rack1       Up     Normal  127.32 KB       66.69%            -9080806731732761568
    172.17.0.3  rack1       Up     Normal  127.32 KB       66.69%            -9017721528639019717
    ...

在这里，我们看到，对于每个令牌，它显示了节点的地址，它在哪个机架上，状态（“上”或“下”），状态，负载和令牌。 “拥有者”列显示该节点实际处理
的环（密钥空间）的百分比。

    $ nodetool describering <keyspace>

显示给定键空间的令牌范围。在三节点群集上的输出看起来像这样：

    Schema Version:082bce63-be30-3e6b-9858-4fb243ce409c
    TokenRange:
    TokenRange(start_token:9143256562457711404, end_token:9156964624790153490, endpoints:[172.17.0.4], rpc_endpoints:[172.17.0.4], endpoint_details:[EndpointDetails(host:172.17.0.4, datacenter:datacenter1, rack:rack1)])
    TokenRange(start_token:9081892821497200625, end_token:9111351650740630104, endpoints:[172.17.0.4], rpc_endpoints:[172.17.0.4], endpoint_details:[EndpointDetails(host:172.17.0.4, datacenter:datacenter1, rack:rack1)])
    ...


我们还可以通过以下方式获取有关集群的信息：

    $ nodetool describecluster
    
    
    Cluster Information:
       Name: Test Cluster
       Snitch: org.apache.cassandra.locator.SimpleSnitch
       Partitioner: org.apache.cassandra.dht.Murmur3Partitioner
       Schema versions:
          082bce63-be30-3e6b-9858-4fb243ce409c: [172.17.0.2, 172.17.0.3, 172.17.0.4]
      
Copyright

© 2016, The Apache Software Foundation.
@ 2020, gottingen lothar
Apache®, Apache Cassandra®, Cassandra®, the Apache feather logo and the Apache Cassandra® Eye logo are either registered
 trademarks or trademarks of the Apache Software Foundation in the United States and/or other countries. No endorsement 
 by The Apache Software Foundation is implied by the use of these marks.