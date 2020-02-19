# Choose a Compaction Strategy


Scylla实施以下压缩策略以减少读取放大，写入放大和空间放大，这会导致瓶颈和性能下降。这些策略包括：

* Size-tiered compaction strategy (STCS) - 当系统具有足够大小（默认为四个）大小相似的SSTables时触发。
* Leveled compaction strategy (LCS)  - 系统使用分布在不同级别的小型固定大小（默认为160 MB）的SSTable。
* Incremental Compaction Strategy (ICS)  - 
与STCS具有相同的读取和写入放大因子，但是它通过将巨大的sstable分解为SSTable运行来解决其2倍的临时空间放大问题，SStable运行由一组较小的
（默认为1 GB），不重叠的SSTables组成。
* Time-window compaction strategy (TWCS) - 设计用于时间序列数据；替换了日期分层压缩
* Date-tiered compaction strategy (DTCS)  -  为时间序列数据而设计。


本文档介绍了如何选择压缩策略，并介绍了每种策略的优缺点。如果您希望获得有关压缩的一般信息或任何这些策略的更多信息，请参阅压缩概述。如果要说
明用于创建压缩策略的CQL命令，请参阅“压缩CQL参考”。

## Size-tiered Compaction Strategy (STCS)


大小分层压缩策略（STCS）的前提是合并大小大致相同的SSTable。

## Size-tiered compaction benefits


这是LSM工作负载的流行策略。这样会导致SSTables的数量低且为对数（数据大小），并且在压缩期间复制相同的数据的次数很少。使用哪种策略最好的表
来确定这是否是满足您需求的正确策略。

## Size-tiered compaction disadvantages


此策略具有以下缺点（尤其是写入）：

* 不断修改现有行会导致每一行被分成多个SSTable，从而使读取速度变慢，这在“分层压缩”中不会发生。
* 在非常大的SSTable中，过时的数据（被覆盖或删除的列）会保留很长时间，浪费空间，直到最终合并为止。例如，在覆盖密集型负载中，开销可能高达
400％，因为数据将在一个层中重复4倍。另一方面，输出SSTable将是单个输入SSTable的大小。结果，您将需要5倍的空间量（4个输入SSTables加上1个
输出SSTable），因此比当前存储的数据量大400％。随着数据集大小的增加，必须检查和评估分配的空间。

* 压缩需要大量的临时空间，因为在清除重复项之前会写入新的更大的SSTable。在最坏的情况下，多达一半的磁盘空间需要为空才能发生这种情况


实施此策略
Set the parameters for [Size-tiered compaction](/scylla/docs/cql/compact.md).

## Leveled Compaction Strategy (LCS)

Leveled Compaction Strategy  使用固定大小的小型SSTable（默认为160 MB）划分为不同的级别。每个级别代表多个SSTable的运行。

## Leveled Compaction benefits


使用 leveled compaction 策略时，以下好处值得注意：

* SSTable读取是有效的。大量的小型SSTables并不意味着我们需要在许多SSTables中查找关键字，因为我们知道每个级别的SSTable都有不相交的范围，
因此我们只需要在每个级别中查看一个SSTable。在典型情况下，只需要读取一个SSTable。
* 使该压缩策略有效的其他因素是，过时的行最多浪费10％的空间，并且仅需要保留约10倍的SSTable小尺寸的空间，以便临时使用压缩。

## Leveled Compaction disadvantages


这种方法的缺点是写入时的I/O多两倍，因此对于专注于主要写入新数据的工作负载而言，它的效果不佳。


一次只能在同一表上执行一次压缩操作，因此，如果已经在进行压缩，则可以推迟压缩。由于文件的大小不是太大，所以这实际上不是问题。


## Incremental Compaction Strategy (ICS)


2019.1.4版中的新功能：Scylla Enterprise

    
    NOTE
    
    ICS仅适用于Scylla Enterprise客户
    

ICS的操作原理类似于STCS的原理，只是用越来越长的SSTable运行替换了每层中越来越大的SSTable，并按照LCS运行建模，但默认情况下使用1 GB的较
大片段。

当有两个或多个大致相同大小的运行时，将触发压缩。这些运行相互增量压缩，从而生成新的SSTable运行，而一旦处理并压缩了输入运行中的每个SSTable，
则逐渐释放空间。通过将开销限制为每个分片两倍（恒定）的片段大小，此方法消除了STCS的高临时空间放大问题。

## Incremental Compaction Strategy benefits

* 大大减少了STCS典型的临时空间放大，导致有更多磁盘空间可用于存储用户数据。
* 鉴于该操作可以以产生新碎片的大致相同的速率释放碎片，因此使用ICS进行大型压实的空间需求几乎不存在。如果您查看此快照，绿线将显示在发布大型压缩
时磁盘使用情况在ICS下的行为。

ncremental Compaction Strategy disadvantages

 
 Namely:
 
* 不断修改现有行会导致每一行被分成多个SSTable，从而使读取速度变慢，这在“分层压缩”中不会发生。
* 过时的数据（被覆盖的列或已删除的列）可能会在各个层上积累很长时间，浪费空间，直到最终合并为止。可以通过不时运行大型压缩来缓解这种情况。

## Time-window Compaction Strategy (TWCS)

Cassandra 3.0.8中引入了用于时间序列数据的时间窗口压缩策略，以替代日期分层压缩策略（DTCS）。时间窗口压缩策略使用大小分层压缩策略（STCS）
在每个时间窗口内压缩SSTable。来自不同时间窗口的SSTable永远不会压缩在一起。在使用CQL命令创建表时，可以设置
TimeWindowCompactionStrategy参数。


    CAUTION
    
    如果使用TWCS，为获得最佳效果，请不要使用多个TTL设置。创建具有混合TTL的多个表将使内容在不同的时间到期，从而导致SSTables无法删除的情
    况。

## Time-window Compaction benefits

* 根据时间范围保留条目，使搜索指定范围内的数据变得容易，从而提高读取性能
* 相对于DTCS进行了改进，将数量减少为大型压实
* 允许一次使整个SSTable到期（使用TTL），因为数据已经在一个时间范围内进行了组织

## Time-window Compaction deficits

* 时间窗口压缩仅适用于时序工作负载

## Date-tiered Compaction Strategy (DTCS)

日期分层压缩是为时间序列数据设计的。 Cassandra 2.1引入了此策略。它仅适用于时间序列数据。不建议使用此策略，而已将其替换为 
Time-window compaction.

## Which strategy is best

每种工作负载类型可能不适用于每种压缩策略。不幸的是，您的工作负载越杂，选择正确的策略就越难。下表显示了根据所指示的工作负荷所使用的策略可以
预期的结果，使您可以做出更明智的决策。请记住，我们测试的最佳选择可能不是您环境的最佳选择。您可能需要尝试找出最适合您的策略。

## Compaction Strategy Matrix


该表列出了哪种工作负载最适合哪种压缩策略。如果您有能力使用STCS或ICS，请始终选择ICS。

![matrix](/scylla/images/matrix.png)


以下注释使用以下缩写描述了每种压缩策略在每个用例上创建的放大类型：

* SA - Size Amplification
* WA - Write Amplification
* RA - Read Amplification

* 将大小分层与仅写负载一起使用时，它将使用大约2倍的峰值空间-具有增量的SA，SA少得多
* 在仅写入负载下使用分层压缩时，您会遇到较高的写入放大-WA
* 当使用累累大小或增量加载时，会发生SA
* 当使用“分层压缩”和覆盖负载时，会发生WA
* 当使用大小分层且读取负荷大且更新很少时，会发生SA和RA
* 当对大多数读取负载进行多次更新时使用Leveled时，WA发生过多
* 将大小分层或增量与时间序列工作负载一起使用时，会发生SA，RA和WA。
* 当使用“按时间序列调整级别”工作负载时，会发生SA和WA。


