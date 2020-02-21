# Scylla Memory Usage

Scylla内存使用量可能大于所使用的数据集。

例如：

    The data size is 19GB, but Scylla uses 220G memory.

Scylla使用可用内存来缓存您的数据。 Scylla知道如何动态管理内存以获得最佳性能，例如，如果许多客户端连接到Scylla，它将从缓存中逐出一些数据
以为这些连接腾出空间，当连接计数再次下降时，该内存将返回到缓存中。

要限制内存使用量，可以使用--memory参数启动scylla。或者，您可以使用--reserve-memory参数指定ScyllaDB应该留给操作系统的内存量。请记住，
留给操作系统的内存量需要足以在JVM之上运行的外部scylla模块，例如scylla-jmx。

On Ubuntu, edit the /etc/default/scylla-server.

On Red Hat / CentOS, edit the /etc/sysconfig/scylla-server.

例如：

    SCYLLA_ARGS="--log-to-syslog 1 --log-to-stdout 0 --default-log-level info --collectd-address=127.0.0.1:25826 \
    --collectd=1 --collectd-poll-period 3000 --network-stack posix --memory 2G --reserve-memory 2G