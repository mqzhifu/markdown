# promehtues报警规则

节点的运行时间

> time\(\) \- node\_boot\_time\_seconds{}

系统15分钟负载

> node\_load1

//==========cpu

节点的cpu 百分比\(system\)

> \(avg by \(environment,instance\) \(irate\(node\_cpu\_seconds\_total{job="node\_206",mode="system"}\[5m\]\)\)\) \* 100

//=======内存

节点的内存总量

> node\_memory\_MemTotal\_bytes{job="node\_206"}

节点的剩余内存量

> node\_memory\_MemFree\_bytes{job="node\_206"}

节点的已使用内存量

> node\_memory\_MemTotal\_bytes{job="node\_206"} \- node\_memory\_MemFree\_bytes{job="node\_206"}

节点的内存使用百分比

> \(\(node\_memory\_MemAvailable\_bytes{job="node\_206"} / \(node\_memory\_MemTotal\_bytes{job="node\_206"}\)\)\)\* 100

节点的内存剩余百分比

> \(1\-\(node\_memory\_MemAvailable\_bytes{job="node\_206"} / \(node\_memory\_MemTotal\_bytes{job="node\_206"}\)\)\)\* 100

//=======磁盘

节点的磁盘总量

> node\_filesystem\_size\_bytes{job="node\_206" ,fstype=~"ext4|xfs"}

节点的磁盘剩余空间

> node\_filesystem\_avail\_bytes{job="node\_206",fstype=~"ext4|xfs"}

节点的磁盘使用的空间

> node\_filesystem\_size\_bytes{job="node\-exporter",fstype=~"ext4|xfs"} \- node\_filesystem\_avail\_bytes{job="node\-exporter",fstype=~"ext4|xfs"}

节点的磁盘的使用百分比

> \(1 \- node\_filesystem\_avail\_bytes{job="node\-exporter",fstype=~"ext4|xfs"} / node\_filesystem\_size\_bytes{job="node\-exporter",fstype=~"ext4|xfs"}\) \* 100

//=====网络

节点当前established的个数

> node\_tcp\_connection\_states{job="node\-exporter", state="established"}

节点timewait的连接数

> node\_tcp\_connection\_states{job="node\-exporter", state="time\_wait"}

节点tcp连接总数

> sum by \(environment,instance\) \(node\_tcp\_connection\_states{job="node\-exporter"}\)

节点网卡eth0每秒接收的比特数

> avg by \(environment,instance,device\) \(irate\(node\_network\_receive\_bytes\_total{device=~"eth0|eth1|ens33|ens37"}\[1m\]\)\)

节点网卡eth0每秒发送的比特数

> avg by \(environment,instance,device\) \(irate\(node\_network\_transmit\_bytes\_total{device=~"eth0|eth1|ens33|ens37"}\[1m\]\)\)

//================================================

实例丢失

up{job="node\-exporter"} == 0

summary: "服务器实例 {{ $labels.instance }} 丢失"

description: "{{ $labels.instance }} 上的任务 {{ $labels.job }} 已经停止了 1 分钟已上了"

磁盘容量小于 5%

100 \- \(\(node\_filesystem\_avail\_bytes{job="node\-exporter",mountpoint=~"._",fstype=~"ext4|xfs|ext2|ext3"} \* 100\) / node\_filesystem\_size\_bytes {job="node\-exporter",mountpoint=~"._",fstype=~"ext4|xfs|ext2|ext3"}\) \> 95

summary: "服务器实例 {{ $labels.instance }} 磁盘不足 告警通知"

description: "{{ $labels.instance }}磁盘 {{ $labels.device }} 资源 已不足 5%, 当前值: {{ $value }}"

内存容量小于 20%

\(\(node\_memory\_MemTotal\_bytes \- node\_memory\_MemFree\_bytes \- node\_memory\_Buffers\_bytes \- node\_memory\_Cached\_bytes\) / \(node\_memory\_MemTotal\_bytes \)\) \* 100 \> 80

summary: "服务器实例 {{ $labels.instance }} 内存不足 告警通知"

description: "{{ $labels.instance }}内存资源已不足 20%,当前值: {{ $value }}"

CPU 平均负载大于 4 个"

node\_load5 \> 4

sumary: "服务器实例 {{ $labels.instance }} CPU 负载 告警通知"

description: "{{ $labels.instance }}CPU 平均负载\(5 分钟\) 已超过 4 ,当前值: {{ $value }}"

磁盘读 I/O 超过 30MB/s"

irate\(node\_disk\_read\_bytes\_total{device="sda"}\[1m\]\) \> 30000000

sumary: "服务器实例 {{ $labels.instance }} I/O 读负载 告警通知"

description: "{{ $labels.instance }}I/O 每分钟读已超过 30MB/s,当前值: {{ $value }}"

磁盘写 I/O 超过 30MB/s"

irate\(node\_disk\_written\_bytes\_total{device="sda"}\[1m\]\) \> 30000000

sumary: "服务器实例 {{ $labels.instance }} I/O 写负载 告警通知"

description: "{{ $labels.instance }}I/O 每分钟写已超过 30MB/s,当前值: {{ $value }}"

网卡流出速率大于 10MB/s"

\(irate\(node\_network\_transmit\_bytes\_total{device\!~"lo"}\[1m\]\) / 1000\) \> 1000000

sumary: "服务器实例 {{ $labels.instance }} 网卡流量负载 告警通知"

description: "{{ $labels.instance }}网卡 {{ $labels.device }} 流量已经超过 10MB/s, 当前值: {{ $value }}"

CPU 使用率大于 90%"

100 \- \(\(avg by \(instance,job,env\)\(irate\(node\_cpu\_seconds\_total{mode="idle"}\[30s\]\)\)\) \*100\) \> 90

sumary: "服务器实例 {{ $labels.instance }} CPU 使用率 告警通知"

description: "{{ $labels.instance }}CPU 使用率已超过 90%, 当前值: {{ $value }}
