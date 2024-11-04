PingCAP 公司自主设计
开源分布式关系型数据库

TiDB 作为 SQL 层，采用 Go 语言开发， TiKV 作为下边的分布式存储引擎，采用 Rust 语言


- TiDB 是 Server 计算层，主要负责 SQL 的解析、制定查询计划、生成执行器。
- TiKV 是分布式 Key-Value 存储引擎，用来存储真正的数据，简而言之，TiKV 是 TiDB 的存储引擎。
- PD 是 TiDB 集群的管理组件，负责存储 TiKV 的元数据，同时也负责分配时间戳以及对 TiKV 做负载均衡调度。
>TiFlash 列存储引擎


TiDB 兼容MYSQL语法

数据采用多副本存储， Multi-Raft


docker pull pingcap/tidb:latest
docker pull pingcap/tikv:latest
docker pull pingcap/pd:latest

```
docker run -d --name pd1 \
-p 2379:2379 \
-p 2380:2380 \
-v /etc/localtime:/etc/localtime:ro \
-v /Users/clarissechamley/data/tidb:/data \
pingcap/pd:latest \
--name="pd1" \
--data-dir="/data/pd1" \
--client-urls="http://0.0.0.0:2379" \
--advertise-client-urls="http://192.168.9.126:2379" \
--peer-urls="http://0.0.0.0:2380" \
--advertise-peer-urls="http://192.168.9.126:2380" \
--initial-cluster="pd1=http://192.168.9.126:2380"
```


```
docker run -d --name tikv1 \
-p 20160:20160 \
--ulimit nofile=1000000:1000000 \
-v /etc/localtime:/etc/localtime:ro \
-v /Users/clarissechamley/data/tidb:/data \
pingcap/tikv:latest \
--addr="0.0.0.0:20160" \
--advertise-addr="192.168.9.126:20160" \
--data-dir="/data/tikv1" \
--pd="192.168.9.126:2379"
```

```
docker run -d --name tidb \
-p 4000:4000 \
-p 10080:10080 \
-v /etc/localtime:/etc/localtime:ro \
pingcap/tidb:latest \
--store=tikv \
--path="192.168.9.126:2379"
```

http://127.0.0.1:2379/dashboard/#/signin