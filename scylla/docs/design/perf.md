# Using the perf utility with Scylla 

本文包含与Scylla一起使用perf实用程序的有用提示和技巧列表。

由于其每核线程的性质，查看聚合很少有用，因为它倾向于隐藏特定于特定CPU的不良行为。查看单个CPU将使这些异常更容易发现。一旦您确定Scylla碎片的行为方式值得研究（例如，通过在特定节点上通过Scylla Monitor碎片视图注意到某个特定碎片比其他碎片加载得更多），就可以使用seastar-cpu-这里描述的map.sh脚本可确定哪个Linux CPU承载有问题的Scylla分片。例如：

    seastar-cpu-map.sh -n scylla -s 0

    shard: 0, cpu: 1

在此示例中，我们要使用-s参数并引用分片编号来研究分片0。结果表明，分片0在Linux CPU 1上运行。在随后的所有perf命令中，可以添加-C 1参数以限制perf只查看CPU 1。

##  When is perf useful

当所探查的CPU以100％的利用率运行时，Perf最为有用，因此我们可以识别出特定功能使用的大量执行时间。

请注意，由于轮询，即使没有瓶颈，Scyla也会轻松将CPU驱动到100％。它将旋转（轮询）一段时间以等待新请求。这往往会在性能报告中显示为与具有较高CPU时间的轮询相关的功能。

当我们怀疑某些本不应该运行的东西正在运行时，Perf也是一种有用的工具。一个示例是具有很高的react__utilization（指示非轮询工作）的系统，在Linux上，system CPU利用率也很高。这表明Linux内核而不是Scylla是CPU的主要用户，因此需要进行其他调查。

## perf top

Perf顶部显示系统的时间点视图。下图是运行perf top -C 1的结果

![perf-top](/scylla/images/perf-top.png)

## Callgraphs

Perf也可以生成调用图。当我们想了解导致调用函数的完整调用链时，它们很有用。我们可以在任何录音命令中添加--call-graph = dwarf -F 99，例如perf top或perf record来生成调用图。

例如，要在CPU2中记录调用图：

    sudo perf record -C 2 --call-graph=dwarf  -F 99

通过运行以下命令，我们可以将结果转储到名为trace.txt的文本文件中：

    sudo perf report  --no-children --stdio > /tmp/trace.txt

