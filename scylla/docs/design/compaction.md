# compaction

 本文档从总体上概述了压缩以及压缩的工作原理。有单独的文档涵盖了用于设置压缩策略的CQL语法，以及涵盖了如何确定最佳策略的压缩策略矩阵。
 
## How Scylla Writes Data
 
 Scylla的写入路径遵循众所周知的日志结构合并（LSM）设计，以实现可立即用于读取的高效写入。 Scylla不是第一个使用此方法的项目。使用此方法的
 热门项目包括Lucene搜索引擎，Google BigTable和Apache Cassandra。Scylla将其更新写入内存表（MemTable），当它变得太大时，它将刷新到新
 文件。对该文件进行排序以使其易于搜索和以后合并。这就是为什么这些表被称为“排序字符串表”或“ SSTables”的原因。
 
 ![compaction-1](/scylla/images/compaction-datawrites.png)
 
 随着时间的流逝，出现了两个主要问题。首先，一个SSTable中的数据随后在另一个SSTable中被修改或删除，这会浪费空间，因为两个表都存在于系统中。
 其次，当将数据拆分到许多SSTable中时，由于需要读取许多SSTable，因此读取请求的处理速度较慢。 Scylla通过使用Bloom过滤器和其他技术来避免从
 不包含所需分区的SSTable中读取，从而缓解了第二个问题。但是，随着SSTables数量的增加，不可避免的是，每个读取查询中需要从中读取的磁盘块的数
 量也会增加。由于这些原因，只要积累了足够的SSTable，Scylla就会执行压缩。
 
 ## Compaction Overview
 
 压缩将几个SSTable合并为新的SSTable，这些SSTable仅包含来自输​​入SSTable的实时数据。合并多个排序的文件以获得排序的结果是一个高效的过程，
 这是保持SSTables排序的主要原因。
 
 
压缩有两种类型：

* Minor Compaction
    

    Scylla根据压缩策略（如下所述）自动触发某些SSTable的压缩。这是推荐的方法。
    
* Major Compaction


    用户触发（使用nodetool）对所有SSTable进行压缩，并根据选择的压缩策略合并各个表。
    

CAUTION
    
    
    始终最好让Scylla自动运行较小的压缩。大型压缩会浪费资源，增加运营成本并占用宝贵的磁盘空间。这要求您的磁盘空间比数据大50％，除非您使用
    增量压缩策略（ICS）。
    

## View Compaction Statistics

Scylla具有可用于查看压缩状态的工具。其中包括nodetool（compacthistory和compactionstats）和Grafana仪表板，它们是Scylla Monitoring 
Stack的一部分，可按每个群集和每个节点显示压缩统计信息。压缩错误可以在日志中看到。

## Compaction strategy


压缩策略是确定要压缩哪个SSTable的时间以及何时压缩。可以使用以下压缩策略，下面将对其进行详细说明。有关将每种策略与其工作量进行比较的矩阵，
请参阅 Compaction Strategy Matrix

* Size-tiered compaction strategy (STCS) - (default setting) 当系统具有足够大小相似的SSTables时触发。

* Leveled compaction strategy (LCS) - the system uses small,固定大小（默认为160 MB）的SSTable分为不同的级别，并降低了读取和
空间放大率。

* Incremental compaction strategy (ICS) -适用于企业客户，以与LCS相似的方式使用排序的，固定大小（默认为1 GB）的SSTables运行，并按
    大小层进行组织，类似于STCS大小层。如果您是企业客户，则ICS是一种旨在替代STCS的更新策略。它具有相同的读写放大率，但由于临时空间开销的减少而
    降低了空间放大率，将其降低到恒定的可管理水平。

* Time-window compaction strategy (TWCS) - 为时间序列数据设计并按时间顺序放置数据。该策略取代了日期分层压缩。 TWCS使用STCS来防止在
    尚未关闭的窗口中累积SSTable。当窗口关闭时，TWCS致力于将时间窗口中的SSTables减少为一个。
    
* Date-tiered compaction strategy (DTCS) - 设计用于时间序列数据，但应改用TWCS。

## How to Set a Compaction Strategy


创建或更改表时，将压缩策略设置为CREATE或ALTER语句的一部分。有关详细信息，请参考CQL语法。


    CAUTION

    更改压缩策略的参数或从一种策略更改为另一种策略（使用ALTER语句）会产生问题。有关更多信息，请参见更改压缩策略或属性。
    

## Size-tiered Compaction Strategy (STCS)

SizeTieredCompactionStrategy（STCS）的前提是合并大小大致相同的SSTable。根据它们的大小，所有SSTable都放入不同的存储桶中。如果
SSTable的大小在参数bucket_low和bucket_high内，则将SSTable添加到现有存储桶中，该参数基于计算存储桶中已经存在的SSTables的当前平均大小。

这将创建多个存储桶，并且当达到存储桶中的表的阈值数量（min_threshold）时，将压缩该存储桶中的表。压缩之后，将表合并，从而产生一个更大的
SSTable。随着时间的推移以及几个大型SSTable的积累，它们将合并形成一个更大的SSTable，依此类推。

这意味着系统具有多个大小层/存储桶（小的SSTable，大的SSTable，甚至更大的SSTable），并且在每个层中，SSTable的数量大致相同。当一层已满（
已达到阈值）时，系统将合并所有表以创建一个SSTable，该SSTable大致属于下一个大小层。

![conmpaction-size](/scylla/images/compaction-size-tiered.png)


如本博客所述，如果分区只写入一次且从未修改过（或写入了几次然后又没有修改过），则SizeTieredCompactionStrategy非常有用。在这种情况下，每
个分区最终最终将整体写入一个压缩的SSTable中，并且读取效率很高。但是，当将STCS与不断修改现有产品的工作负载一起使用时，每个分区都拆分为多个
SSTable，从而使读取速度变慢。分层压缩策略（LCS）不会发生这种情况。

## Leveled Compaction Strategy (LCS)

分层压缩使用分成固定级别的小型固定大小（默认为160 MB）的SSTable。每个级别代表多个SSTable的运行。

## A run of SSTables¶


 运行是将大型SSTable拆分为几个较小的SSTable的日志结构合并（LSM）术语。换句话说，运行是具有不重叠键范围的SSTable的集合。运行的好处是，
 在执行压缩后，仅压缩它的一部分（小的SSTables）并删除。压缩之后，SSTables变小了，不需要一次压缩一个巨大的SSTable。
 
![comapction-leveld](/scylla/images/compaction-leveled.png)


压缩方法的工作方式如下:

1. 在级别0中创建新的SSTable（从MemTables创建）。所有其他级别都是SSTable的运行，其大小呈指数增长，如下所示：
    1. ***level 1***是10个SSTable的运行（每个表160 MB * 10）
    2. ***level 2***是运行100个SSTables（每个表160 MB * 100），等等。
    
2. 当级别0中有4个SSTable时，将使用级别1中的10个SSTable进行压缩。此压缩的工作方式如下：
    * 并行读取级别0的4个SSTable和级别1的10。
    * 为级别1编写新的SSTables（替换已压缩的10个旧表）。
    * 代替创建一个大的SSTable，而是如下编写几个SSTable：创建一个SSTable。当达到大小限制（160 MB）时，将启动一个新表。当数据在已排序键
    上合并时，这将生成一个具有不重叠键范围的运行（请参阅SSTable的运行）。
    
3. 如果从级别0压缩到级别1后，如果级别1的SSTable超过10个，则将级别1多余的SSTable压缩并放入级别2，如下所示：
    * 从级别1中获取一个SSTable（压缩后将删除此SSTable）
    * 查看此SSTable的键范围，并找到第2层与其重叠的所有SSTable。通常，其中大约有12个（级别1的SSTable大约占键的1/10，而每个级别2的
    SSTable大约占键的1/100，因此10个级别2的SSTable将与级别1的SSTable的范围重叠，再加上边缘多两个）。
    * 与以前一样，压缩1级的1 SSTable和2级的12 SSTable，并在2级创建新的SSTable（并删除1 + 12原始SSTable）。
    * 如果在将级别1压缩为级别2之后，在级别2中有多余的SSTables（因为级别2只能容纳100个表），请将它们合并到级别3中。
    
## Temporary Fallback to STCS

当非常快地写入新数据时，“分层压缩”策略可能暂时无法满足需求。这会导致L0中大量SSTable的积累，进而导致读取速度非常慢，因为所有读取请求都是
从L0中的所有SSTable中读取的。因此，作为紧急措施，当L0中的SSTables数量增加到32时，LCS会退回到STCS，以快速减少L0中的SSTables数量。最终，
LCS会将这些数据再次移到更高级别的固定大小的SSTables中。

同样，当引导新节点时，SSTables从其他节点流式传输。保持远程SSTable的级别是为了避免多次压缩，直到完成引导程序为止。在引导过程中，新节点在从
远程节点流传输数据时会收到常规的写入请求。像任何其他写操作一样，这些写操作将刷新到L0。如果Scylla在这些L0 SSTable上执行了LCS压缩并在更高
级别上创建了SSTable，则可能阻止了远程SSTable进入正确级别（请记住，运行中的SSTable不能具有重叠的键范围）。为了纠正这种情况，Scyla仅在L0
中使用STCS压缩表，直到引导过程完成为止。完成后，所有恢复都将在LCS下照常进行。

## Incremental Compaction Strategy (ICS)


2019.1.4版中的新功能：Scylla Enterprise


    NOTE
    
    ICS仅适用于Scylla Enterprise客户
    
大小分层压缩的问题之一是它需要临时空间，因为SSTables在完全压缩之前不会被删除。 ICS采用不同的方法，将每个大型SSTable按与LCS相同的方式，
分成一组排序的，固定大小（默认为1 GB）的SSTable（也称为片段），但它将整个运行而不是将单个SSTable视为STCS的大小调整文件。由于运行片段很
小，因此SSTables可以快速压缩，从而可以在压缩后立即删除各个SSTables。这种方法使用少量的内存和临时磁盘空间。


ICS使用与STCS相同的原理，其中SSTables根据其大小在存储桶中排序。但是，与STCS不同，ICS压缩使用SSTable运行作为输入，并生成新的运行作为输
出。一次运行是否仅由一个可能来自STCS迁移的片段组成并不重要。从增量压缩的角度来看，一切都是运行。

![sompaction-inream](/scylla/images/compaction-incremental.png)

该策略的工作原理如下：

1. ICS会寻找大小相似的压缩候选对象。这些候选者称为输入运行。
    * 每个输入运行可能包含一个或多个SSTable。

2. ICS将两个或多个相似大小的输入运行压缩为一个输出运行

3. 增量压缩一次渐进地作用于两个或多个片段，每个输入运行一次。

    * 它从所有输入片段中读取突变，然后将它们合并到一个输出片段中。
    * 只要结果片段小于sstable_size_in_mb，就不需要采取进一步的措施。
    * 如果片段大于sstable_size_in_mb：
       * 达到大小阈值时停止，然后密封输出片段。 
       * 创建一个新的运行片段，并继续压缩其余的输入片段，直到达到大小阈值为止。
       * 输入片段用尽后，将其从SSTables列表中删除以进行压缩，然后将其从磁盘中删除。
       * 重复直到没有输入片段剩下。
4. 提取所有输出片段，并在SSTable运行时将其反馈回压缩。
5. 当输入运行中的所有片段都用尽并释放时，请停止。

    
    NOTE
    
    
    为了防止在压缩过程中scylla崩溃时数据复活，除了包含实时数据的输出运行外，ICS还可能编写一个包含可清除墓碑的辅助运行。这些文件保留在磁
    盘上，而SSTables包含它们表示墓碑可能会遮盖的数据。压缩完成后，从所有SSTable中删除所有阴影数据，然后清除可清除的逻辑删除并将保存它们
    的SSTable从存储中删除。
    
## Time-window Compaction Strategy (TWCS)¶

引入了时间窗口压缩策略，以替代用于处理时间序列工作负载的日期分层压缩策略（DTCS）。时间窗口压缩策略使用大小分层压缩策略（STCS）在每个时间
窗口内压缩SSTable。来自不同时间窗口的SSTable永远不会压缩在一起。


    CAUTION
    
    
    如果使用TWCS，为获得最佳效果，请不要使用多个TTL设置。创建具有混合TTL的多个表将使内容在不同的时间到期，从而导致SSTables无法删除的情况。
    

该策略的工作原理如下:

1. 配置了时间窗口。该窗口由压缩窗口大小compaction_window_size和时间单位（compaction_window_unit）确定。

2. 使用大小分层压缩策略（STCS）压缩在时间窗口内创建的SSTable。
3. 时间窗口结束后，获取在该时间窗口中创建的所有SSTable，并将数据压缩到一个SSTable中。
4. 最终生成的SSTable永远不会与其他时间窗口的SSTable一起压缩。

通过这种解释，如果时间窗口是一天，那么在一天结束时，仅将当天累积的SSTable压缩为一个SSTable。

## When time-series data gets out of order

TWCS的主要动机是按时间戳分隔磁盘上的数据，并允许完全过期的SSTables更有效地删除。当数据不按顺序写入SSTable时，
新数据和旧数据位于同一SSTable中时，效率会停止。乱序数据可以通过两种方式出现在同一SSTable中：

* 如果用户在传统的写入路径中混合了旧数据和新数据，则数据将被混合到MemTables中，并刷新到同一SSTable中，并保持混合状态。
* 如果用户对旧数据的读取请求导致读取修复，从而将旧数据拉入当前的MemTable。数据在MemTables中混合并刷新到相同的SSTable中，在该SSTable中
将保持混合状态。


在TWCS尝试最小化混合数据的影响时，用户应尝试避免这种行为。具体来说，用户应避免使用显式设置时间戳的查询。建议运行频繁的修复（以不会混入数据
的方式传输数据），并通过将表的read_repair_chance和dclocal_read_repair_chance设置为0来禁用后台读取修复。

## Date-tiered compaction strategy (DTCS)


日期分层压缩是为时间序列数据设计的。它仅适用于时间序列数据。此策略已由时间窗口压缩策略（TWCS）代替。


日期分层压缩策略的工作方式如下：

* 首先，它按时间对SSTable进行排序，然后压缩相邻（按时间）的SSTable
* 这导致SSTables的大小随着年龄的增长而呈指数增长。

例如，在某个时间点，我们可以在一个SSTable中拥有最后一分钟的数据（默认情况下，base_time_seconds = 60），在另一SSTable中拥有另一分钟，
然后在一个SSTable中拥有最后4分钟，然后在那之前有4分钟，然后是之前16分钟的SSTable，依此类推。这种结构可以很容易地通过压实来保持，这与尺寸
分层压实非常相似。当有4个（min_threshold的默认值）小的（一分钟）连续SSTables时，它们被压缩为一个4分钟的SSTable。当有四个更大的SSTable
依次（按时间顺序）时，它们将合并为一个16分钟的SSTable，依此类推。

不会压缩早于max_SSTable_age_days（默认为365天）的旧式SSTable，因为对大多数查询而言，执行这些压缩将无用，处理过程将非常缓慢，并且压缩将
需要大量的临时磁盘空间。

## Changing Compaction Strategies or Properties

## Changing the Threshold in LCS

在某些情况下，可能会在以下级别创建压缩表，该级别在经过相当长的时间后不会被压缩。例如，用户具有使用LCS的表。当前有5个表级别，
SSTable_size_in_mb为5MB。用户将此阈值更改为160MB。进行此更改之后，只有足够的数据才能在同一节点上实际获得L3。 L4中的SSTables中的数据
将被饿死，并且不会被压缩。为了避免这种情况，LCS尝试在以后的压缩中包括那些饥饿的高级SSTables。如果经过25次压实后未对某个级别进行压实，则将
其带入下一次压实。

## Changing to Time Window Compaction Strategy (TWCS)¶


如果要对现有数据启用TWCS，则可以考虑首先运行主要压缩，将所有现有数据放入一个（旧）窗口中。随后的后续写入操作将按预期创建典型的SSTables。


## Changing the Time Window in TWCS


如果要更改时间窗口，可以这样做，但是请记住，由于相邻窗口连接在一起，此更改可能会触发其他压缩。如果窗口大小减小（例如，从24小时减少到12小时
、），则不会修改现有的SSTables。还要注意，TWCS不能将现有的SSTables拆分为多个窗口。

## Which Strategy is best to use¶


使用[Choose a Compaction Strategy](/scylla/docs/architecture/compaction-strategy.md)来确定适合您需求的策略。
