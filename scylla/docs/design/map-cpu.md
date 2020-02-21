# Map CPUs to Scylla Shards


由于其基于内核的线程体系结构，因此基于每个CPU查看Scylla中的许多内容可以更好地理解它。有Linux工具，例如top和perf，可以在给定CPU编号的情况下提供有关CPU内部发生的情况的信息。

用户常犯的一个错误是假设Scylla Shard ID与CPU ID之间存在直接且可预测的关系，这是不正确的。

从3.0版开始，Scylla附带了一个脚本，可让用户了解CPU与Scylla碎片之间的映射。对于旧版本的用户，可以从以下位置下载脚本的副本：[searstar](https://raw.githubusercontent.com/scylladb/seastar/master/scripts/seastar-cpu-map.sh)

## Examples of usage

列出特定分片的映射：

    seastar-cpu-map.sh -n scylla -s 0
    shard: 0, cpu: 1

列出所有分片的映射：

    $ seastar-cpu-map.sh -n scylla
    shard: 0, cpu: 1
    shard: 1, cpu: 2
    shard: 3, cpu: 4
    shard: 2, cpu: 3
