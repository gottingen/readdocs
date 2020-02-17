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

在CQL表定义中， primary key 子句至少指定一个单列分区键，并且还可以指定聚簇键列。 primary key唯一地标识表中的每个分区/行组合，而聚类键指示在给定分区内如何对数据（行）进行排序。有关更多信息，请参见我们的CQL文档。

![table](/scylla/images/table-1.png)


