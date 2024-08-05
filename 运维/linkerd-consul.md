# linkerd-consul

## 相关文档

官方安装文档

> https://www.bookstack.cn/read/linkerd/doc\-getting\-started\-locally.md

官方配置文档

> https://api.linkerd.io/head/linkerd/index.html\#introduction

finagle文档

> https://twitter.github.io/finagle/guide/Clients.html\#load\-balancing

国内版学习文档

> https://gitchat.csdn.net/columnTopic/5a3757cfd7fd136499739b8e

先安装 JDK

## 概览

linkerd简单理解：就是一个代理器，有点类似网关（但不是真的网关）

![WX20211202-125716@2x.png](image/WX20211202-125716@2x.png)

LINKERD的作用：

1. 基于感知时延的负载均衡
2. 运行时动态路由
3. 熔断机制、流量控制
4. 服务发现管理
5. 多协议兼容

注意：说它不是真的传统型网关，因为它是作用在client以sidecar模式运行，也就是在client端开一个linkerd进程守护

缺点：因其运行在client端，更适合后端内部使用

> l5d：linkerd 缩写,其中5:inker，代理5个字母

内部模块流转

|名称      |描述                                                          |
|----------|--------------------------------------------------------------|
|Identifier|根据配置规则，识别出请求的service\-name                       |
|dtab      |delegation table/委托表的简称                                 |
|bindding  |根据service\-name去dtab找寻条目，进行 绑定，找到服务发现解释器|
|Resolution|根据service\-name 解析出一个 IP:PORT                          |

  

```mermaid

graph LR

request-->Identification

Identification-->service-name

service-name-->dtab

dtab-->clientname

clientname-->resolution

resolution-->replicas

```

配置模块说明

|名称   |描述                  |
|-------|----------------------|
|admin  |管理                  |
|namers |通过哪种方式，服务发现|
|routers|路由的详细配置        |
|       |                      |

下载安装包

> wget https://github.com/linkerd/linkerd/releases/download/1.6.0/linkerd\-1.6.0.tgz

解压

> tar \-xvf linkerd\-1.6.0.tgz

添加配置文件

```
cd /linkerd-1.6.0/config
touch config/linkerd.yaml
vim config/linkerd.yaml
```

这里先用一个最简单，先确定测试跑起来

```
admin:
  ip: 0.0.0.0
  port: 9990

namers:
- kind: io.l5d.fs
  rootDir: disco

routers:
- protocol: http
  dtab: |
    /svc => /#/io.l5d.fs;
  httpAccessLog: logs/access.log
  label: int
  servers:
  - port: 4140
    ip: 0.0.0.0
```

阿里云打开3个端口

> 9990 WEB\-UI管理界面
> 
> 
> 4140 http & 4141 grpc

添加一个服务IP:PORT,做测试

```
cd disco
touch test1
vim test1
127.0.0.1 7771
```

随便找个进程软件（apache nginx），改下端口号，开启监听7771

启动

> ./linkerd config/linkerd.yaml

启动加日志

> \-log.level=ALL \-log.output=all.log

查看下可视化，是否正常

> http://8.142.177.235:9990

metrics

> http://39.107.127.244:9990/admin/metrics.json

测试，发一个请求

> curl \-H 'Host:test1' http://127.0.0.1:4140

我们要请求服务名为：test1

通过设置HTTP的header的host=test1

1. linkerd接收到这个请求，会去disco目录下的test1文件中的配置的IP:PORT
2. 将请求转发这个IP:PORT，这里配置的是7771端口号
3. 将返回结果再返回给调用方

上面在请求过程中，是将header中的host设定为service名称

现在改一下curl 中 \-H 的Host 这个key

> curl \-H "test\-header:test1" http://127.0.0.1:4140

修改如下配置文件即可

```
- protocol: http
//添加下面两行
identifier:kind: io.l5d.header
header：test-header
```

另外，不光光一定要用header做service name识别，下面还有几种方法：

1. method
2. post get
3. path

应用场景不同，使用的类型不同

## namerd

以上服务发现\(IP:PORT\)都是 file\-base ，通过配置文件，来实现服务发现\-代理，不可动态变更,namerd就是直接将请求转向3方，如：ETCD CONSUL

官方配置文档

> https://api.linkerd.io/1.1.2/namerd/index.html\#introduction

下载包：namerd namerctl

> https://github.com/linkerd/namerctl/releases/download/0.8.6/namerctl\_linux\_amd64

先配置namerd

touch /soft/linkerd/config/namerd.yaml

```
#后台UI管理
admin:
  ip: 0.0.0.0
  port: 9991

#所有的配置读取完后，缓存在内存中
storage:
  kind: io.l5d.inMemory
namers:
- kind: io.l5d.fs
  rootDir: disco
interfaces:
- kind: io.l5d.thriftNameInterpreter
  port: 4100
  ip: 0.0.0.0
- kind: io.l5d.httpController
  ip: 0.0.0.0
  port: 4321
```

启动namerd

> ./namerd config/namerd.yaml

端口：4100 4321 9991

> 测试下，打开可视化管理
> 
> 
> http://8.142.177.235:9991/

能显示，但是会报错

修改linkerd

注销掉如下两行

```
#dtab: |
#  /svc => /#/io.l5d.fs;
```

添加两行：

```
 interpreter:
    kind: io.l5d.namerd
    dst: /$/inet/127.0.0.1/4100
    namespace: web
```

重启linkerd

创建一个新的dtab配置文件

> touch /soft/linkerd/config/dtab\-config

设置一条路由，将所有请求转向fs处理

> /svc =\> /\#/io.l5d.fs;

现在，动态创建一个规则：

> ./namerctl dtab create web config/dtab\-config \-\-base\-url http://127.0.0.1:4321

打开可视化，查看下刚刚添加的web

> http://8.142.177.235:9991/

测试一下

curl \-H "Host:test1" http://127.0.0.1:4140

至此，基础的工作。。。都讲完了，虽然各种细节还没有

## consul

下载安装包

> https://releases.hashicorp.com/consul/1.8.0/consul\_1.8.0\_linux\_amd64.zip

> unzip consul\_1.8.0\_linux\_amd64.zip
> 
> 
> mv consul /soft

启动

> ./consul agent \-dev \-ui \-node=consul\_node\_dev \-client=0.0.0.0

阿里云 开 8500

> http://8.142.177.235:8500

优雅关闭consul

> ./consul leave

创建读取服务的目录

> mkdir consul.d

注册一个服务:以配置文件的方式

```
cd consul.d
touch test.json
vim test.json
{"service":{"id": "test","name": "test","address": "127.0.0.1","port": 80,"tags": ["dev"]}}
```

重新启动consul

> ./consul agent \-dev \-ui \-node=consul\-dev \-client=0.0.0.0 \-config\-dir consul.d \-pid\-file=/tmp/consul.pid

查看下刚刚注册的服务

> http://8.142.177.235:8500

动态 HTTP 注册

> curl \-X PUT \-d '{"id": "test2","name": "test","address": "127.0.0.1","port": 80,"tags": \["dev"\]}' http://127.0.0.1:8500/v1/agent/service/register

服务发现

> curl localhost:8500/v1/catalog/service/test

```
{
    "service":{
        "id":"test",
        "name":"test",
        "address":"39.107.127.244",
        "port":80,
        "tags":[
            "dev"
        ],
        "checks":[
            {
                "http":"http://127.0.0.1:80/",
                "tls_skip_verify":false,
                "method":"Get",
                "interval":"10s",
                "timeout":"1s"
            }
        ]
    }
}
```

---

简单科普下consul

agent client server

集群

注册service

发现service

8301:lan gossip

8302:wlan gossip

8500:http UI admin

8501：httpsapi端口，默认disabled

8502:grpc,默认disabled

8600:DNS

server\-\>Raft\-\>server

client\-\>Gossip\-\>client

client\-\>grpc\-\>server

默认启动：是server模式，也就是启动参数加了 \-server=true

node\-name:在集群中的，节点名称，需要唯一。默认走本地的hostname

Datacenter：数据中心，默认是dc1\(Segment: '\<all\>'\)

## namerd连接consul

linkerd配置文件不需要用

创建 namerd\-consul.yml

```
namers:
- kind: io.l5d.consul
  host: 127.0.0.1
  port: 8500
```

启动

> ./namerd config/namerd\-consul.yaml

创建 config/dtab\-config\-consul

```
/svc => /#/io.l5d.consul/dc1;
```

生成一个新的路由条目

./namerctl dtab create web config/dtab\-config\-consul \-\-base\-url http://127.0.0.1:4321

测试linkerd日志

> cat /tmp/linkerd.log |awk '{print $1}'|sort |uniq \-c
