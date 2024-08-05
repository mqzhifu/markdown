机械硬盘

一个硬盘，是由若干块磁片（CD）组成，每个磁片上有个磁头，磁盘是竖向叠加在一起，磁头也一样。于是一块硬盘内部原理就是：由若干磁片组成的一个 圆柱体。

每个磁片，又被分散成几块，叫：扇区。

超级块：硬盘最开始的一块区别，存储该硬盘 文件文件系统信息，如：当前硬盘是以EXT3格式格式的，容量总大小、被分了几个区等。

INODE：超级块后面第个区域就是INODE区域，保存一些跟文件相关的汇总索引信息。

数据：第3个区域全是真正的数据信息。

扇区（sector）:硬件（磁盘）上的最小的操作单位,是操作系统和块设备（硬件、磁盘）之间传送数据的单位。

block由一个或多个sector组成，文件系统中最小的操作单位；OS的虚拟文件系统从硬件设备上读取一个block，实际为从硬件设备读取一个或多个sector。

实际为从硬件设备读取一个或多个sector。对于文件管理来说，每个文件对应的多个block可能是不连续的。因为你无法预知一个文件有多少个block，还有在做 IO操作时，会产生碎片 。

也就是说，一个文件，会占用多个不连续的硬盘block。

既然文件的数据都存于block中，那么还得有个地方存元数据。INODE产生。

VFS

虚拟文件系统 Virtual File System：内核暴给用户态应用层的接口。

inode : index node ，索引节点

inode_cache：文件元数据，如：文件真实物理位置、文件大小、内存映射、文件创建UID GID、访问权限、访问时间、修改时间、创建时间

//文件管理、创建、删除、重命名、创建链接

directory_cache

page_cache

buffer_cache

device drivers

proc sysfs ramfs

ext2 ext3 ext4

VFS 在以上所有中间，在用户态之下。类似一个适配器。

dentry：directory entry 缩写

Superblock

每启动一个进程，都会有一个 ' 文件描述符-表'，名为：file table。里面是所有该进程打开的所有文件。在file table结构体中维护File Status Flag（file结构体的成员f_flags）和当前读写位置(file结构体的成员f_pos)。file table 里的每一项都是指向file struct 结构体，其里面的某一项，会指向：file opertions 结构体，这个结构体里是：read write 。假设现在A B 进程同时打开 X文件，在其file table中就都有X 文件描述符，但最后指向file opertions 结构体是一个，当然会有ref_count做区别。

open 一个文件的时候，传入的是一个路径，需要根据路径找inode，内核缓存了dentry。

file table -> file opertion dentry->inode->inode_operations|super_block