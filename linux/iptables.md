# iptables

规则链

input 所有以主机为目的地的

output 所有源自主机的

forward 这些请求不是主机，路经主机（路由）

满足第一条规则，后面的规则就会再被执行

SNAT：内网主机访问外网而经过路由时，源 IP 会发生改变

DNAT：如上，反之，目标IP 发生改

\-A, \-\-append

iptables \-A INPUT

\-I, \-\-insert

\-p, \-\-protocol

\-\-dport, \-\-destination\-port

\-m multiport \-\-source\-port 22,53,80,110

\-m multiport \-\-destination\-port 22,53,80,110

\-m multiport \-\-port 22,53,80,110

\-m limit \-\-limit 3/hour

\-m mac \-\-mac\-source 00:00:00:00:00:01

\-m state \-\-state

INVALID 表示该封包的联机编号（Session ID）无法辨识或编号不正确。

ESTABLISHED 表示该封包属于某个已经建立的联机。

NEW 表示该封包想要起始一个联机（重设联机或将联机重导向）。

RELATED 表示该封包是属于某个已经建立的联机，所建立的新联机。例如：FTP\-DATA 联机必定是源自某个 FTP 联机

\-j 参数用来指定要进行的处理动作，ACCEPT、REJECT、DROP、REDIRECT、MASQUERADE、LOG、DNAT、SNAT、MIRROR、QUEUE、RETURN、MARK

作用于传输层

数据包从外网传送到防火墙后，防火墙抢在IP层向TCP层传送前，将数据包转发给包检查模块进行处理

首先与第一个过滤规则比较，如果与第一个模块相同，则对它进行审核，判断是否转发该数据包，这时审核结果是转发数据包，则将数据包发送到TCP层进行处理，否则就将它丢弃。

iptables基础知识：

ptables由表 \(tables\)，链\(chains\)，规则\(rules\)组成

默认iptables内置了四个表（tables）：filter表 nat表 mangle表和raw表

五个链：INPUT ,OUTPUT ,PREROUTING ,POSTROUTING ,FORWARD

filter表:INPUT ,OUTPUT ,FORWARD

nat表:PREROUTING ,POSTROUTING, OUTPUT

mangle表:ALL（跟路由无关，在任何点都行，ye最高）

raw表：

当一个数据包进入网卡时，它首先进入PREROUTING链，内核根据数据包目的IP判断是否需要转送出去。

如果数据包就是进入本机的，它就会沿着图向下移动，到达INPUT链。数据包到了INPUT链后，任何进程都会收到它。本机上运行的程序可以发送数据包，这些数据包会经过OUTPUT链，然后到

达POSTROUTING链输出。

如果数据包是要转发出去的，且内核允许转发，数据包就会如图10\-4所示向右移动，经过FORWARD链，然后到达POSTROUTING链输出。

格式：

表 操作表的指令 匹配 动作

iptables \[\-t table\]\( commands\[ chain rule num \]\) match criteria \-j TARGET

最简单的例子：

iptables \-A INPUT \-p tcp \-\-dport 80 \-j ACCEPT

操作 表 匹配 动作

仅允许内网网问80端口，（0/24表示：1\-255）

iptables \-A INPUT \-s 192.168.0.0、24 \-p tcp \-\-dport 80 \-j ACCEP

开户本地访问\(lo:loopback,环回，当访问127.0.0.1 localhost,实际不走 eth0,而是走lo接口\)

iptables \-A INPUT \-i lo \-j ACCEPT

iptables \-A OUTPUT \-i lo \-j ACCEPT

COMMANDS

CHAIN: 对链进行的操作

\-N：new 新建一条链

\-X 删除一条用户自定义链（空链）

\-F：flush 清空一条链，默认清空表中所有链

\-Z：zero 清空计数器，iptables中每条规则默认有两个计数器，用于记录本条规则所匹配到的数据包的个数和本条规则所匹配到的数据包的总大小

\-P：policy 定义链的默认处理策略

\-E 重命名链

RULE：对规则进行的操作

\-A:append 追加，在链的最后加一条规则

\-I:insert 插入一条规则 一般使用\-I CHAIN NUM 给规则加一个编号。

\-R:replace 替换某条规则，规则被替换并不会改变顺序，必须要指定替换的规则编号：\-R CHAIN NUM。

\-D:delete 删除一条规则，可以输入完整规则，或者直接指定标号加以删除：\-D CHAIN NUM。

辅助性子命令：\-n numeric 以数字的形式来显示地址，默认显示主机名称

\-v verbose 显示详细信息 ，支持\-vv \-vvv格式，v越多，信息越详细。

\-x 显示原有信息，不要做单位换算

\-\-line\-numbers 显示规则的行号

Match Creteria（匹配规则）：

基本匹配

\-s，\-\-src，\-\-source 匹配数据包的源地址

\-d，\-\-dst，\-\-destination 匹配数据包的目标地址

\-i， 指定数据包的流入接口（逻辑接口）

\-o， 指定数据包的流出接口

\-p， 做协议匹配 protocol，（tcp|udp|icmp）

扩展匹配：对某一种功能的扩展

隐含扩展 ：对某一种协议的扩展

\-p tcp

\-\-sport 指定源端口

\-\-dport 指定目的端口

\-\-tcp\-flags（SYN，ACK，FIN，PSH，URG，RST，ALL，NONE）指定TCP的标志位

需要跟两个标志位列表，如：SYN，ACK，FIN，RST SYN 第一个列表表示要检查的位，第二个

列表表示第一个列表中出现的位必须为1，未出现的必须为0

\(注意：\-\-sport、\-\-dport 必须联合 \-p 使用）

\-\-syn 只允许新连接

\-p udp 无连接协议

\-\-sport 指定源端口

\-\-dport 指定目的端口

\-p icmp

\-\-icmp\-type echo\-request，8（ping出去，请求回应，） echo\-reply，0（给予回应）

显式扩展 \(附加模块\)：额外附加的更多的匹配规则,功能性地扩展

\-m state 状态检测扩展

NEW 用户发起一个全新的请求

ESTABLISHED 对一个全新的请求进行回应

RELATED 两个完整连接之间的相互关系，一个完整的连接，需要依赖于另一个完整的连接

INVALID 无法识别的状态

\-m multiport \-\-sports 22,80,443 指定多个源端口

\-\-dports 22，80,443 指定多个目标端口

\-\-ports 非连续端口

\-m connlimit 限定并发连接速率

```
         ！--connlimit-above 5  高于五个将拒绝
 -m string   字符串匹配
         --algo bm|kmp 指定算法 
         --string pattern
 -m time  基于时间的匹配
    --timestart
    --timestop
    --days
```

\-j TARGET 处理动作\(只支持大写\)

```
 ACCEPT    接受
 DROP      悄悄丢弃，请求端没有任何回应
 REJECT    明确拒绝
 SNAT      源地址转换
 DNAT      目标地址转换
 REDIRECT  端口重定向
 LOG       将访问记录下来
```
