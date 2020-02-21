# Debug your database with Flame Graphs


Flame Graphs用作调试工具，以识别延迟和占用大部分CPU时间的执行路径部分。在以下情况下使用火焰图：

* 需要了解使用最多时间的Scylla代码路径/函数。例如，当您遇到延迟问题时。
* 需要比较花费在不同碎片上的特定Scylla代码路径/功能上的时间。例如，当您在一个CPU上遇到延迟问题，而在另一个CPU上却没有。

## Run a Flame Graph

***Procedure***

1. Clone the repoistory

    
        git clone https://github.com/brendangregg/FlameGraph
2. 转到Flamegraph软件包所在的目录。
   
        cd FlameGraph

3. 运行以下perf命令，将Map CPU映射到Scylla Shards，并将perf实用程序与Scylla结合使用，以供参考。

        sudo perf record --call-graph dwarf -C <CPU on which you are onrecording>
        sudo perf script | `pwd`/stackcollapse-perf.pl | `pwd`/flamegraph.pl > some_name.svg
    
4. 结果是一个.svg文件，它不仅是图片，而且是一个动态图，您可以在其中搜索，放大和缩小。为了正确享受Flame Graph的所有功能，您可以在Chrome或Firefox等浏览器中将其打开。

![flame](/scylla/images/flamegraph.svg)

## Tips

* 在要记录的CPU上，尝试加载Scylla以消耗100％的CPU运行时间。否则，您会看到很多与空闲时间处理相关的操作系统功能
* 在所有分片上记录（例如，使用“ perf record” -p参数）可能导致记录从不同线程（分片）调用的同一符号的结果令人困惑。不建议这样做。


