# CPU/内存管理

## CPU

SMP的：对称多处理（Symmetrical Multi\-Processing）技术，在一个计算机上汇集了一组处理器\(多CPU\),各CPU之间共享内存子系统以及总线结构。

3种 模型

均匀存储器存取（Uniform\-Memory\-Access，UMA）：所有CPU共享，均匀使用内存。

非均匀存储器存取（Nonuniform\-Memory\-Access，NUMA）

只用高速缓存的存储器结构（Cache\-Only Memory Architecture，COMA）

这些模型的区别在于存储器和外围资源如何共享或分布。

具体没时间研究，我大概的理解是：可以聚集一组CPU，在某一机器上计算，可以访问本机内存，也可以访问远方内存，有点云计算的意思。

另外，还有CPU 上的硬盘 缓存 的东东，空了再研究吧。
