# Operating System (OS) Support Guide

Red Hat Enterprise Linux版本7和更高版本，CentOS版本7和更高版本，Ubuntu 14.04和16.04 LTS和Debian 8支持Scylla。Scylla也可以在其他
发行版和版本上构建，但未在其上进行测试。我们建议使用支持的发行版之一进行生产。

由于Scylla严重依赖XFS，因此RHEL或CentOS优于Ubuntu。这是因为RHEL和CentOS非常注重XFS，从而提高了性能并更及时地提供了错误修复程序。

我们建议使用操作系统的最新更新，以避免已知的错误或安全漏洞。

## OS Support Matrix

    NOTE
    
    对于此表中列出的所有OS版本，该术语及以上仅指次要发行版。
    
OS	|Supported?	|Recommended?
:---|:---|:---
RHEL 7.2 and above	| yes| yes
RHEL 7.0 - 7.1| yes|
CentOS 7.2 and above| yes | yes	
CentOS 7.0 - 7.1| yes		 
Oracle Linux 7.2 and above	| yes	 
Ubuntu 14.04 LTS| yes
Ubuntu 16.04 LTS| yes	 
Ubuntu 18.04 LTS *	| yes	 
Debian 8.6 and above| yes	 
Debian 9.0 *| yes

Supported in Scylla 2.

## References

* [Red Hat Enterprise Linux 7](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/index.html)
* [CentOS 7](https://www.centos.org/download/)
* [Ubuntu 14.04 LTS](http://releases.ubuntu.com/14.04/)
