# gateway-APISIX

## 初衷/目的

寻找一个开源软件：对外的公共网关

## 原因

因为linkerd没有限量 黑名单等功能，或者说linkerd就不是对外公共网关，他的侧重点更多是服务间的治理。因此，还得找一个API公共网关，看了下主流的：kong kyt APISIX Zuul

Zuul首先淘汰，JAVA体系里的，1是重 2是尝试捆绑JAVA

kyt：GOLANG编写，但是高级功能收费

kong:绑定postgresql DB,不支持GRPC WEEBSOCKET MQTT,对服务发现也不太友好

最后意外发现了个：APISIX

## 需求

1. 服务发现与治理,consul
2. 负载均衡
3. 熔断
4. 限制，速度 请求数 并发
5. 黑名单\(ip/uid\)
6. 支持http https grpc websocket
7. 验证，JWT OAUTH
8. 链路追踪 ，zipKin
9. 可视化后台管理工具，同时支持HTTP管理
10. 日志，prometheus
11. 兼容旧代码，代码侵入

## 核心需求

1. 防攻击
2. 服务发现与治理

## APISIX

官方文档

> https://apisix.apache.org/zh/docs/apisix/getting\-started

中文文档

> https://www.bookstack.cn/read/apache\-apisix\-1.5\-zh/a2c94bb1dcdc1a83.md

英文文档

> https://apisix.apache.org/docs/apisix/discovery/consul\_kv

其实它有点是KONG的升级版，底层依然是：NGINX\+OPENRESTY，只是放弃了postgreSql，换成了ETCD，同时优化了下结构，代码更少，对一些主流的软件支持略友好，且扩展自由度更高

> 具说性能比KONG至少高2倍以上.

这个软件好像是2019.6月才开源，稳定性对比kong 应该略差些，毕竟，KONG运行了好多年。不过，2021年的我们，再用版本已经到了2.11 应该免去了这个麻烦。

## 普通安装

官方文档用的是docker，因为我是测试，就换了普通的yum方式

先看一下依赖包：open\-restry ,etcd

> 仅限 centos7

折腾一圈下来，遇到各种问题，最大问题是：open\-restry 兼容性

etcd也得有修改点参数：

> enable\-v2: true \#如果用V2版本，使用HTTP的方式，得打开，不然404
> 
> 
> enable\-grpc\-gateway: true

正式安装 apisix

官方用的是yum，3方是下包

> wget https://downloads.apache.org/apisix/2.11.0/apache\-apisix\-2.11.0\-src.tgz

tar \-zxvf

mv

添加lua依赖库

> make deps
> 
> 
> LUAROCKS\_SERVER=https://luarocks.cn make deps

报错，得安装 luarocks

> yum install \-y luarocks lua\-devel lualdap

重装执行，开始下载了。。。时间还挺长.....

报错，得安装ldap

> yum update

> yum \-y install openldap compat\-openldap openldap\-clients openldap\-servers openldap\-servers\-sql openldap\-devel

好了....

执行，lua http 库存有问题，放弃了，改用docker吧

## 再次尝试

> yum \-y install yum\-utils

openrestry

> yum install \-y https://repos.apiseven.com/packages/centos/apache\-apisix\-repo\-1.0\-1.noarch.rpm

> yum\-config\-manager \-\-add\-repo https://repos.apiseven.com/packages/centos/apache\-apisix.repo

> sudo yum install apisix

终于成功了。。。。

对比了下他的几个依赖包

```
apisix-base                    x86_64           1.19.3.2.2-0.el7                 release              35 M
 openresty-openssl111           x86_64           1.1.1l-1.el7                     openresty           1.5 M
 openresty-pcre                 x86_64           8.44-1.el7                       openresty           164 k
 openresty-zlib                 x86_64           1.2.11-3.el7.centos              openresty            54 k

```

默认安装在/usr/local下

apisix start

apisix test

apisix init

apisix init\_etcd

\#优雅停止

apisix quit

\#强制停止

apisix stop

瞬间启动成功... 真不知道前天折腾到3点图啥...

本机再安装一个nginx 80 和 apache 8088 用于测试

先创建一个Upstream

```
curl http://127.0.0.1:9080/apisix/admin/upstreams/10 -X PUT -H "X-API-KEY: edd1c9f034335f136f87ad84b625c8f1" -d '
{
  "type": "roundrobin",
  "nodes": {
    "127.0.0.1:80": 1,
    "127.0.0.1:8088": 1
  }
}'
```

查看

> curl http://127.0.0.1:9080/apisix/admin/upstreams \-H "X\-API\-KEY: edd1c9f034335f136f87ad84b625c8f1"

创建router并绑定upstream

```
curl "http://127.0.0.1:9080/apisix/admin/routes/1" -H "X-API-KEY: edd1c9f034335f136f87ad84b625c8f1" -X PUT -d '
{
  "uri": "/apisix/",
  "host": "service1",
  "upstream_id": "10"
}'
```

验证

> curl \-i \-X GET "http://127.0.0.1:9080/apisix/" \-H "Host: service1"

做下基础验证的测试

创建一个consumer

```
curl http://127.0.0.1:9080/apisix/admin/consumers -H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' -X PUT -d '
{
    "username": "service1_user",
    "plugins": {
        "key-auth": {
            "key": "auth-one"
        }
    }
}'
```

修改路由

```
curl "http://127.0.0.1:9080/apisix/admin/routes/1" -H "X-API-KEY: edd1c9f034335f136f87ad84b625c8f1" -X PUT -d '
{
    "plugins": {
        "key-auth": {
            "header": "ser_key"
        }
    },
  "uri": "/apisix/",
  "host": "service1",
  "upstream_id": "10"
}'
```

> curl \-i \-X GET "http://127.0.0.1:9080/apisix/" \-H "Host: service1" \-H "ser\_key:auth\-one"

还有：jwt base\-auth 不测了，有点鸡肋，主要浪费网关计算时间

## docker安装

将 Apache APISIX 的 Docker 镜像下载到本地

> git clone https://github.com/apache/apisix\-docker.git

将当前的目录切换到 apisix\-docker/example 路径下

> cd apisix\-docker/example

运行 docker\-compose 命令，安装 Apache APISIX

> docker\-compose \-p docker\-apisix up \-d

还是官方文档，docker 安装靠谱，分分钟搞定了...

看了下它的编排文件，里面包含的东西还挺多

1. nginx\(两个\) 9081 9082
2. apisix 9080 9091 9443 9092
3. prometheus 9090
4. etcd 2379
5. grafana 3000
6. apisix\-dashboard 9000

停止

> > docker\-compose \-p docker\-apisix stop

测试一下

> curl "http://127.0.0.1:9080/apisix/admin/services/" \-H 'X\-API\-KEY: edd1c9f034335f136f87ad84b625c8f1'

看一下dashboard

> http://8.142.177.235:9000/

## 初步测试

创建一个upstreams

```
curl "http://127.0.0.1:9080/apisix/admin/upstreams/1" -H "X-API-KEY: edd1c9f034335f136f87ad84b625c8f1" -X PUT -d '
{
  "type": "roundrobin",
  "nodes": {
    "httpbin.org:80": 1
  }
}'
```

创建一个router,并绑定到上面的upstreams上

```
curl "http://127.0.0.1:9080/apisix/admin/routes/1" -H "X-API-KEY: edd1c9f034335f136f87ad84b625c8f1" -X PUT -d '
{
  "uri": "/get",
  "host": "httpbin.org",
  "upstream_id": "1"
}'
```

验证

> curl \-i \-X GET "http://127.0.0.1:9080/get?foo1=bar1&foo2=bar2" \-H "Host: httpbin.org"

## 数据流转

req\-\>etcd\-\>router\-\>plugin\-\>filter\-\>filter\-plugin\-\>upstream

\-\>plugin\-\>real\-service

核心：router upstream plguins

我们看实际就是通过plugins将router upstream串联起，最后找到后端服务器

## router

功能：通过 req信息，找到，router，再 找到upstream

先根据用户的请求信息，如：header uri ，去ETCD里匹配，找到一个router，然后根据该条router的配置信息，执行插件，最后找到一个upstream

plugin

|key                 |desc                              |type    |
|--------------------|----------------------------------|--------|
|echo                |                                  |        |
|grpc\-transcode     |                                  |        |
|jwt                 |                                  |验证    |
|key\-auth           |                                  |验证    |
|basic\-auth         |                                  |验证    |
|cros                |跨域                              |        |
|ip\-restriction     |                                  |安全    |
|ua\-restriction     |过滤UA的，可以过滤掉一些爬虫      |安全    |
|referer\-restriction|                                  |安全    |
|limit\-count        |根据时间窗口内，限制请求次数      |安全    |
|limit\-req          |根据时间窗口内，请求频率          |        |
|limit\-conn         |根据时间窗口内，限制连接数\(并发\)|        |
|ip\-restriction     |IP 黑/白名单                      |        |
|api\-breaker        |熔断                              |        |
|traffic\-split      |分流                              |流控    |
|request\-id         |                                  |        |
|prometheus          |开放HTTP接口等待对方来抓metrics   |监控统计|
|zipkin              |链路追踪，得配合request\-id       |监控统计|
|log\-rotate         |对本地磁盘日志文件动态切割        |日志    |
|tcp\-logger         |将日志主动推送3方                 |日志    |
|traffic\-split      |分流                              |流控    |

关于安全，可以将几个合起来使用：

单个节点，并发不能超10个，每秒最多60次请求，10秒最多500个请求，且UA正常，不在IP黑名单里，且key\-auth正常

基本上能过滤掉所有初级的黑客的DDOS攻击了，另外就是粒度能小到UID 就完美了（不过网关层也不应该关注UID）

验证插件：

略有点特殊，他得绑定一个consumer，并不是应用层理解的，JWT转出UID，所以，它的作用更像是：不允许所有请求都可以访问网关，算是个初级防御吧。综合看下来：key\-auth更适合些

插件

“remote\_addr”

“server\_addr”

“http\_x\_real\_ip”

“http\_x\_forwarded\_for”

对于IP做安全有个问题：

他是挂在ROUTER上的，那么，如果一个服务后面挂了4台机器，等于这个限流是直接针对4台机器的了。。。

limit\-req

限制请求速率

1秒内只可以访问一次，如果大于1 小于3会被延迟，如果大于3直接reject

同样也是基于IP，跟上面有相同问题

api\-breaker：配合check\-health，错误次数，熔断若干时间，最后达到 阀值直接断了，但是不确定check\-health的方式，是针对一组服务的所有机器，还是某个机器。

traffic\-split：可以做灰度发布，但还没研究明白

## upstream

一个服务器配置\(ip:port\)集合表，或者理解为：一组服务的集群

负载均衡也是在这里做的，目前支持如下几种

|key        |dsec                                          |
|-----------|----------------------------------------------|
|roundrobin |轮询，带权重                                  |
|chash      |一致性哈希                                    |
|ewma       |选择延迟最小的节点                            |
|least\_conn|选择 \(active\_conn \+ 1\) / weight 最小的节点|
|balancer   |用户自定义的                                  |

> 感觉也够用

1. 轮询，带权重
2. 一致性哈希

服务发现

Eureka / Consul

discovery\_type

health\-check

健康检查

有点鸡肋，如果启用服务发现，这功能没啥用。。。

另外，只有有请求后health\-check才会触发

retries

scheme：http https grpc

echo

## 服务发现与治理

Eureka consul dns

dns:忽略，负载策略单一，部署成本高

Eureka:忽略，JAVA的东西

consul:主要分析这个

## 总结

感觉网关还是挺有用的，间接反映以前对网关认识不足，之前很多在应用层做的事情完全都可以抽离出来放在网关层统一处理。

如果对apisix做个简短定义： 就是个 nginx 升级版.

## etcd 开启grpc getway proxy

> ./etcd grpc\-proxy start \-\-endpoints=http://127.0.0.1:2378 \-\-listen\-addr=127.0.0.1:2379
