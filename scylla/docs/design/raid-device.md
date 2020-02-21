# Recreate RAID devices 

Scylla设置的一部分将在分配给它的所有存储设备上创建一个RAID设备。但是，在某些情况下，我们只想重做此步骤，而无需再次调用整个设置阶段。这种情况的一个例子是在具有临时存储的云中使用Scylla。硬停止后，存储设备将被重置，以前的设置将被破坏。要重新创建RAID设备，请运行以下脚本：

    scylla_raid_setup --raiddev /dev/md0 --disks <comma-separated list of disks>

完成此步骤后，将挂载，格式化存储并创建
/var/lib/scylla 