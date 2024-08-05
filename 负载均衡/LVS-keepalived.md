# 概览

Linux Virtual Server：采用 IP 负载均衡技术和基于内容请求分发技术

# LVS 有几种分发模式

## NAT

- 客户端发起请求
- 服务端(Director Server)IPTABLE 接收到请求
- IPTABLES-->PREROUTING 判断客户的请求目标 IP 地址是本地，转 INPUT 链
- IPVS 比对数据包请求的服务是否为集群服务，若是，修改数据包的目标 IP 地址为后端服务器 IP
- 然后将数据包发至 POSTROUTING 链。 此时报文的源 IP 为客户 ID，目标 IP 改为后端服务器 IP
- POSTROUTING 链通过选路，将数据包发送给-后端服务器 IP
- Real Server 比对发现目标为自己的 IP，开始构建响应报文发回给 Director Server。
- 此时报文的源 IP 为本机 IP，目标 IP 为客户 IP
- Director Server 在响应客户端前，此时会将源 IP 地址修改为自己的 VIP 地址
- 然后响应给客户端。 此时报文的源 IP 为 VIP，目标 IP 为 CIP

优点：支持端口转发

缺点：请求/响应，都要经常 负载机

## DR

同上，但换 MAC，不换 IP，最后，返回数据的时候，不走负载机，直接返回客户端

优点：返回数据不用再经常负载机，性能好

缺点：不支持端口映射，网段要一致

## TUN

是在 IP 头部再增加一层，IP

# LVS 由 2 部分程序组成

1.ipvs(ip virtual server)：一段代码工作在内核空间，工作在 INPUT 链上

叫 ipvs，是真正生效实现调度的代码。

2. ipvsadm：另外一段是工作在用户空间，叫 ipvsadm，负责为 ipvs 内核框架编写规则，定义谁是集群服务，而谁是后端真实的服务器(Real Server)

# 十种调度算法，分静/动

## 静态

RR：roundrobin，轮询；

WRR：Weighted RR，加权轮询，性能高的服务器，加权，处理的请求多一点

SH：Source Hashing，实现 session sticy，源 IP 地址 hash；将来自于同一个 IP 地址的请求始终发往第一次挑中的 RS，从而实现会话绑定；

DH：Destination Hashing；目标地址哈希，将发往同一个目标地址的请求始终转发至第一次挑中的 RS，典型使用场景是正向代理缓存场景中的负载均衡

## 动态

lc：least connections，根据连接数，Overhead=activeconns\*256+inactiveconns

WLC：Weighted LC，权重最小连接，Overhead=(activeconns\*256+inactiveconns)/weight 数

lblc：根据请求 IP，寻找距离最近的 IP-SERVER，且该服务器没有超载，否则会根据且连接数分发

lblcr：同上，不同的是本地会维护一个表，一个 IP 到一组 IP 的距离。一般是做 CDN-缓存

SED：Shortest Expection Delay，Overhead=(activeconns+1)\*256/weight

NQ：Never Queue

总结：感觉就是利用 IPTABLE 做了一层代理（IPVS），且在内核态不需要用户介入

# keepalived

最初，是为 LVS 设计，专门用来监控集群系统中各个服务节点的状态。它根据 TCP/IP 参考模型的第三、第四层、第五层交换机制检测每个服务节点的状态，如果某个服务器节点出现异常，或者工作出现故障，Keepalived 将检测到，并将出现的故障的服务器节点从集群系统中剔除，这些工作全部是自动完成的，不需要人工干涉。

后来 Keepalived 又加入了 VRRP 的功能，VRRP（Vritrual Router Redundancy Protocol,虚拟路 由冗余协议)出现的目的是解决静态路由出现的单点故障问题，通过 VRRP 可以实现网络不间断稳定运行，因此 Keepalvied 一方面具有服务器状态检测和故障隔离功能。

防单点故障，如上的 LVS 负载机，如果挂了，就全挂了

VRRP 协议：将多个物理路由器，合并成一个虚拟路由器。虚拟路由器会产生虚拟 IP，并对外服务。同一时间只能有一个路由器处理网络的所有工作，其它的路由器只接收主路由器发送的 PING 包

1、两台机器安装 KEEPALIVED，一台配置中写名 MASTER，另一台标记 BAK。

2、选择一个虚拟 IP，绑定到一个网卡接口，设置虚拟路由 ID 号（两台一样），配置上两台机器的实际 IP。

3、设置权重值，主要高于 BAK，主要两台机器是竞争上岗。

4、当请求过来的时候，会先到虚拟 IP，再到 MASTER。同时，MASTER 间隔向 BAK 发送心中包，BAK 只负责接收，一但接收不到。BAK 就会修改自己为 MASTER，虚拟 IP 就会分发到 BAK 机器。
