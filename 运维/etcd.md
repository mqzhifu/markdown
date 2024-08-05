# etcd

https://github.com/etcd\-io/etcd/releases

解压后，两个主要的文件：

1、etcd
2、etcdctl

先启动守护进程,看下效果

> ./etcd

配置文件

```
mkdir config
cd config
touch et\_m1.yml et\_m2.yml et\_m3.yml
```


数据目录

> mkdir /soft/etcd/data/
> cd /soft/etcd/data/
> mkdir et\_m1 et\_m2 et\_m3
> chmod 777 /soft/etcd/data
> chmod \-R 700 /soft/etcd/data/\*
> vim m1.yml

```

name: "et_m1"
data-dir: "/soft/etcd/data/et_m1"
log-level: "debug"

listen-peer-urls: 'http://172.26.150.7:2380'
listen-client-urls: 'http://0.0.0.0:2379'
initial-advertise-peer-urls: 'http://172.26.150.7:2380'
advertise-client-urls: 'http://0.0.0.0:2379'

initial-cluster-state: 'new'
initial-cluster-token: 'etcd-cluster'

initial-cluster: 'et_m1=http://172.26.150.7:2380,et_m2=http://172.26.150.7:2382,et_m3=http://172.26.150.7:2384'

enable-v2: true #如果用V2版本，使用HTTP的方式，得打开，不然404

```

> 如果间单机，此配置必须得有：listen\-client\-urls: 'http://0.0.0.0:2379'

2379：CLIENT、HTTP API 端口

2380：各集群节点通信端口

启动

./etcd \-\-config\-file=/soft/etcd/config/cfg.conf

重启启动

> cp /soft/etcd/etcd /usr/local/bin
> 
> 
> etcd \-\-config\-file=/soft/etcd/config/et\_m1.yml
> 
> 
> etcd \-\-config\-file=/soft/etcd/config/et\_m2.yml
> 
> 
> etcd \-\-config\-file=/soft/etcd/config/et\_m3.yml

http v2 测试，添加一个数据

> curl http://39.106.65.76:2379/v2/keys/message \-X PUT \-d value="Hello world"

http v2 测试，获取一个数据

> curl http://39.106.65.76:2379/v2/keys/message

etcdctl ，指令行的日常操作，跟HTTP API 一样

> ./etcdctl put test 1111
> 
> 
> ./etcdctl get test

在使用 etcdctl ，有个V2 V3切换

> export ETCDCTL\_API=2
> 
> 
> export ETCDCTL\_API=3

新的版本ETCD均是V3\-API，使用了GRPC，具说性能提升1/3，最可恨的是V3不兼容V2，不能共存，V3：是放弃了HTTP，全用GRPC，如果非要使用HTTP，可以再搞个grpc getway ，用HTTP\+JSON发送过来，中转下

## etcd 添加服务

touch /usr/lib/systemd/system/etcd.service

vim /usr/lib/systemd/system/etcd.service

```
[Unit]
Description=etcd key-value store
Documentation=https://github.com/etcd-io/etcd
After=network-online.target local-fs.target remote-fs.target time-sync.target
Wants=network-online.target local-fs.target remote-fs.target time-sync.target

[Service]
User=root
Type=notify
#Environment=ETCD_DATA_DIR=/soft/etcd/data
#Environment=ETCD_NAME=%m

#WorkingDirectory=/soft/etcd/
#EnvironmentFile=-/soft/etcd/config/et_m1.yml
ExecStart=/soft/etcd/etcd --config-file=/soft/etcd/config/et_m1.yml

Restart=always
RestartSec=10s
LimitNOFILE=40000

[Install]
WantedBy=multi-user.target
```

systemctl enable etcd.service

Created symlink from /etc/systemd/system/multi\-user.target.wants/etcd.service to /usr/lib/systemd/system/etcd.service.

systemctl start etcd.service;

## 安全

查看下先

etcdctl user list

添加用户/设置密码

etcdctl user add root

开启验证功能

./etcdctl auth enable \-\-user='root'

角色与目录权限

这里先忽略掉吧

## 原理

```
Raft：etcd所采用的保证分布式系统强一致性的算法。
Node：一个Raft状态机实例。
Member： 一个etcd实例。它管理着一个Node，并且可以为客户端请求提供服务。
Cluster：由多个Member构成可以协同工作的etcd集群。
Peer：对同一个etcd集群中另外一个Member的称呼。
Client： 向etcd集群发送HTTP请求的客户端。
WAL：预写式日志，etcd用于持久化存储的日志格式。
snapshot：etcd防止WAL文件过多而设置的快照，存储etcd数据状态。
Proxy：etcd的一种模式，为etcd集群提供反向代理服务。
Leader：Raft算法中通过竞选而产生的处理所有数据提交的节点。
Follower：竞选失败的节点作为Raft中的从属节点，为算法提供强一致性保证。
Candidate：当Follower超过一定时间接收不到Leader的心跳时转变为Candidate开始竞选。
Term：某个节点成为Leader到下一次竞选时间，称为一个Term。
Index：数据项编号。Raft中通过Term和Index来定位数据。
```
