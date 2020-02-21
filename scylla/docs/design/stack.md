# Decoding Stack Traces

## Synopsis

本文介绍如何解码Scylla日志中找到的堆栈跟踪。

## What are Stack Traces


由于各种错误或在常规数据库操作过程中，堆栈跟踪可能会出现在日志中。能够解码跟踪以了解到底发生了什么很有用。解码堆栈跟踪需要特定Scylla构建的Debug二进制文件，并已安装正在使用的OS。

请注意，将堆栈跟踪信息作为支持服务单或Github问题的一部分进行共享，可以帮助Scylla支持团队更好地理解问题。

## Install Debug Binary files

根据您的操作系统发行版安装调试二进制文件

## RPM based distributions

### Scylla Open Source

    yum install scylla-debuginfo

### Scylla Enterprise

    yum install scylla-enterprise-debuginfo

## DEB based distributions

### Scylla Open Source

    apt-get install scylla-server-dbg

### Scylla Enterprise

    apt-get install scylla-enterprise-server-dbg

## Validate the debug binary file name

要稍后确定实际的调试二进制文件，请在/usr/lib/debug/目录中查找名为scylla.debug的文件。

例如：

    # find /usr/lib/debug/ | grep scylla.debug
    /usr/lib/debug/usr/bin/scylla.debug

## Locate and Analyze the Logs

***Procedure***

1. 从scylla.debug日志中复制堆栈并将其放在文件中（例如，trace.txt）。


    Jul 14 16:56:50 hostname01 scylla: Aborting on shard 8.
    Jul 14 16:56:50 hostname01 scylla: Backtrace:
    Jul 14 16:56:50 hostname01 scylla: 0x0000000000688602
    Jul 14 16:56:50 hostname01 scylla: 0x00000000005a36fc
    Jul 14 16:56:50 hostname01 scylla: 0x00000000005a39a5
    Jul 14 16:56:50 hostname01 scylla: 0x00000000005a3a53
    Jul 14 16:56:50 hostname01 scylla: /lib64/libpthread.so.0+0x000000000000f6cf
    Jul 14 16:56:50 hostname01 scylla: /lib64/libc.so.6+0x0000000000036276
    Jul 14 16:56:50 hostname01 scylla: /lib64/libc.so.6+0x0000000000037967
    Jul 14 16:56:50 hostname01 scylla: 0x00000000024ce37b
    Jul 14 16:56:50 hostname01 scylla: 0x00000000024d2a47
    Jul 14 16:56:50 hostname01 scylla: 0x00000000024df1d5
    Jul 14 16:56:50 hostname01 scylla: 0x00000000023dccec
    Jul 14 16:56:50 hostname01 scylla: 0x00000000023feac1
    Jul 14 16:56:50 hostname01 scylla: 0x00000000024324b9
    Jul 14 16:56:50 hostname01 scylla: 0x00000000024357e4
    Jul 14 16:56:50 hostname01 scylla: 0x0000000001eace90
    Jul 14 16:56:50 hostname01 scylla: 0x0000000001eaf944
    Jul 14 16:56:50 hostname01 scylla: 0x0000000000584ab6
    Jul 14 16:56:50 hostname01 scylla: 0x0000000000584c71
    Jul 14 16:56:50 hostname01 scylla: 0x00000000006522ff
    Jul 14 16:56:50 hostname01 scylla: 0x00000000006554e5
    Jul 14 16:56:50 hostname01 scylla: 0x000000000073d3cd
    Jul 14 16:56:50 hostname01 scylla: /lib64/libpthread.so.0+0x0000000000007e24
    Jul 14 16:56:50 hostname01 scylla: /lib64/libc.so.6+0x00000000000febac


2. 请执行以下任一操作
   1. 对于Scylla 3.0及更低版本，请下载seastar-addr2line并使其可执行。
   2. 对于Scylla 3.1和更高版本，该脚本是标准安装的一部分。
3. 运行解码器：

    ./seastar-addr2line -e /usr/lib/debug/usr/bin/scylla.debug -f trace.txt > trace_decoded.txt

trace_decoded.txt现在包含堆栈跟踪的解码版本：

例如：

    void seastar::backtrace<seastar::backtrace_buffer::append_backtrace()::{lambda(seastar::frame)#1}>(seastar::backtrace_buffer::append_backtrace()::{lambda(seastar::frame)#1}&&) at /usr/src/debug/scylla-2.3.2/seastar/util/backtrace.hh:56
    seastar::backtrace_buffer::append_backtrace() at /usr/src/debug/scylla-2.3.2/seastar/core/reactor.cc:390
    (inlined by) print_with_backtrace at /usr/src/debug/scylla-2.3.2/seastar/core/reactor.cc:411
    seastar::print_with_backtrace(char const*) at /usr/src/debug/scylla-2.3.2/seastar/core/reactor.cc:418
    sigabrt_action at /usr/src/debug/scylla-2.3.2/seastar/core/reactor.cc:3939
    (inlined by) operator() at /usr/src/debug/scylla-2.3.2/seastar/core/reactor.cc:3921
    (inlined by) _FUN at /usr/src/debug/scylla-2.3.2/seastar/core/reactor.cc:3917
    ...

    seastar::reactor::run_tasks(seastar::reactor::task_queue&) at /usr/src/debug/scylla-2.3.2/seastar/core/reactor.cc:2621
    seastar::reactor::run_some_tasks() at /usr/src/debug/scylla-2.3.2/seastar/core/reactor.cc:3033
    seastar::reactor::run_some_tasks() at /opt/scylladb/include/c++/7/chrono:377
    (inlined by) seastar::reactor::run() at /usr/src/debug/scylla-2.3.2/seastar/core/reactor.cc:3180
