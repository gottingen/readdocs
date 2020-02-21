# system limited


应当设置或删除以下系统限制。

Name	|Required value	|Description
:---|:---|:---
core	|unlimited	|maximum core dump size
memlock	|unlimited	|maximum locked-in-memory address space
nofile	|200000	|maximum number of open file descriptors
as	|unlimited	|address space limit
nproc	|8096	|maximum number of processes

该配置在文件/etc/security/limits.d/scylla.conf.中提供