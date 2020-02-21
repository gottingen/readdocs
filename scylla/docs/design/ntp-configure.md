# NTP configuration for Scylla

Apache Cassandra和Scylla取决于准确的系统时钟。jepsen distributed systems testing tool 的作者[写道](https://aphyr.com/posts/299-the-trouble-with-timestamps):

    Apache Cassandra使用服务器或客户端（可选）提供的挂钟时间戳来订购写入。对于给定的时间戳，它可以保证写入和读取的单调性。例如，Cassandra保证在大多数情况下，如果您成功地写入了一定数量的节点，那么从一定数量的节点中进行的任何后续读取都将看到该写入或带有更大时间戳的写入。

因此，服务器需要保持时间同步。这不是一个难题，因为我们在Linux系统上都拥有NTP，对吗？不完全的。对于独立服务器，NTP开箱即用的方式很好，但是对于分布式数据存储来说可能是个问题。

## You did WHAT in the Pool

典型的Linux系统随附的默认NTP配置使用“ NTP池”，这是由具有公关意识的Internet计时系统管理员提供的公开可用时间服务器的列表。池是一项有价值的服务，但是为了节省任何给定服务器上的NTP流量负载，请使用DNS循环机制对其进行管理。一个尝试解析主机名0.pool.ntp.org的客户端将获得与另一个客户端不同的结果。

正如Viliam Holub在分为两部分的系列文章（第1部分，第2部分）中指出的那样，如果集群中的Apache Cassandra节点独立地从Internet上的随机池服务器获取时间，则两个节点可以通过NTP广泛共享的机会标准）差异时间很高。例如，如果一个群集有10个节点，则某对节点在50％的时间中的时间相差超过10.9ms。该问题只会随着添加更多节点而加剧。


解决方案是能够获取Linux发行版随附的ntp.conf文件，并删除默认的“池”服务器，并放入数据中心自己的NTP服务器。

而不是看起来像这样的行：

    server 0.fedora.pool.ntp.org iburst
    server 1.fedora.pool.ntp.org iburst

或者：

    server 0.debian.pool.ntp.org iburst
    server 1.debian.pool.ntp.org iburst

使用您自己的服务器。因此，ntp.conf将有指向您自己的NTP服务器的“服务器”行，看起来更像是：

    # begin ntp.conf

    # Store clock drift -- see ntp.conf(5)
    driftfile /var/lib/ntp/drift

    # Restrict all access by default
    restrict default nomodify notrap nopeer noquery

    # Allow localhost access and LAN management
    restrict 127.0.0.1
    restrict ::1
    restrict 192.168.1.0 mask 255.255.255.0 nomodify notrap

    # Use our company’s NTP servers only
    server 0.ntp.example.com iburst
    server 1.ntp.example.com iburst
    server 2.ntp.example.com iburst

    # End ntp.conf

可以将相同的ntp.conf部署到数据中心中的所有服务器。不只是Apache Cassandra节点，还有使用它们的应用程序服务器。对于整个群集来说，时间同步比任何节点在互联网上匹配一些随机计算机要重要得多。将数据存储时间与应用程序服务器的时间保持一致也很有帮助，以便于进行故障排除和匹配日志条目。

专用NTP设备可用，对于大型站点而言可能是一个不错的选择。否则，任何标准的Linux系统都应该成为一个好的NTP服务器。

在NTP服务器上，如果没有已知的定时服务器，则可以继续使用Linux发行版附带的“ pool.ntp.org”服务器行。但是，好的托管服务提供商或企业级ISP可能在网络上具有离您很近的NTP服务器，这将是替换池条目的更好选择。

您的NTP服务器应该互相对等：


    peer 0.ntp.example.com prefer
    peer 1.ntp.example.com
    peer 2.ntp.example.com

## Pass the Fudge

网络断开时会发生什么？在大多数情况下，NTP应该可以正常工作。您的NTP服务器之间将建立新的共识时间。老式的NTP文档使用“多余的”行来在网络连接失败时让NTP服务器依靠本地系统时钟。在现代版本的NTP上，“孤立”功能已替换为 [Orphan mode](http://support.ntp.org/bin/view/Support/OrphanMode)。

在每个NTP服务器上的ntp.conf中添加 orphan 行：

    tos orphan 9

如果外部服务器遇到问题，NTP服务器将做正确的事情并保持彼此之间的同步。

仅此而已。一个相对简单的系统管理项目可以在以后节省大量的故障排除麻烦。NTP服务器正常工作后，请自己查看 [instructions for joining the NTP pool](http://www.pool.ntp.org/join.html)，以便您可以与他人共享正确的时间
