# ElasticSearch

官网：https://www.elastic.co/cn/downloads/elasticsearch

添加一个用户，ES不允许ROOT启动

```
groupadd esgroup;
useradd esroot -g esgroup -p esroot
```

下载包，解压~设置权限

> chown \-R esroot:esgroup /soft/elasticsearch
> 
> 
> su \- esroot
> 
> 
> chown \-R esroot:esgroup es
> 
> 
> chmod 777 /data/logs/es

修改个数值，不然无法启动

> max virtual memory areas vm.max\_map\_count \[65530\] is too low, increase to at least \[262144\]

vim /etc/sysctl.conf

vm.max\_map\_count=262144

sudo sysctl \-p

编辑一下配置文件

> vim /soft/elasticsearch/config/elasticsearch.yml

```
cluster.name: my-es-cluster
node.name: node-76
path.data: /data/es
path.logs: /data/logs/es
network.host: 0.0.0.0
http.port: 9200
cluster.initial_master_nodes: ["node-76"]

```

修改个数值，不然无法启动

> vim jvm.options

\-Xmx512m

\-Xmx512m

启动

> su \- esroot
> 
> 
> /soft/elasticsearch/bin/elasticsearch \-d

```
future versions of Elasticsearch will require Java 11; your Java version from [/soft/jdk/jre] does not meet this requirement
```

注：ES里自带有JDK11，所以如果机器没安装JDK，就别装了，费力不讨好。

看下端口是否启动成功

> lsof \-i:9200

restful api 访问

> http://59.110.167.206:9200/

查看集群简单健康状态

> http://8.142.177.235:9200/\_cluster/health?pretty

查看集群状态

> http://8.142.177.235:9200/\_cluster/state

查看集群配置

> http://8.142.177.235:9200/\_cluster/settings

关闭ES

之前还有个HTTP\-API的方式，7.0以后给干掉了，目前没有太好的办法 ，只能KILL

> curl \-XPOST 'http://8.142.177.235:9200/\_shutdown

## ES基础学习

索引\(index\)：间接理解为，DBMS中的一个数据库，可分片、分副本

索引类型\(index\_type\)：间接理解为，DBMS中的，一个表，每个表有不同的字段值

文档\(doc\)：间接理解为，DBMS中的一行记录

映射\(mapping\)：索引和索引类型

> 7.0\-8.0官方已经宣布干掉index\_type，可以用使用：\_doc，做为index\_type name。结果就是：没有索引类型这个，索引下直接挂字段修饰符

> http://ip:port/索引/类型/文档ID 废弃 v:7.0\-
> 
> 
> http://ip:port/索引/\_doc/文档ID 使用 v7.0\+

document:

1. \_index：文档所在的索引名
2. \_type：文档所在的类型名
3. \_id：文档唯一 id
4. \_score：相关性算分
5. \_uid：组合 id，有\_type和\_id组成（从6.x开始\_type不再起作用，同\_id一样）
6. \_source：文档的原始 json 数据，可以从这里获取每个字段的内容
7. \_all：整合所有字段内容到该字段，默认下禁用

> document其实是2个部分，1：元数据，2：用户的真实数据

一个索引的定义主要包含2个KEY：setting mappings

1. setting:修改分片和副本数的
2. mappings:修改字段和类型的，是对一个索引的具体修饰，像是MYSQL的表，不同的是它更灵活。它支持动态创建，换个角度理解：是对doc的约束。

> http://59.110.167.206:9200/索引名/\_setting
> 
> 
> http://59.110.167.206:9200/my\_first\_index/\_mappings

查找索引下的数据

> curl \-XPOST 'http://59.110.167.206:9200/filebeat\-7.13.1\-2021.06.08\-000001/\_search?pretty' \-d '{"query": { "match\_all": {} },"from": 10,"size": 10}' \-H "Content\-Type: application/json"|grep messag

删除一个索引

> curl \-XDELETE 'http://59.110.167.206:9200/filebeat\-7.13.1\-2021.06.08\-000001'

查看索引列表

> http://59.110.167.206:9200/\_cat/indices?format=json&index=\[索引名称，可使用通配符\]

查看索引mapping

> http://59.110.167.206:9200/my\_first\_index/\_mapping?pretty

创建一个索引

```
curl -XPUT http://59.110.167.206:9200/my_first_index -H 'Content-Type: application/json' -d '
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas":0
  }
}'
```

> 创建索引的同时也可以一并创建索引类型

索引成功后，就是索引类型了，其实就是mapping，往一个索引里添加/注册mapping结构体

索引类型

1. type :String Numerical Date bool Array Object等
2. index:true|false ，是否被索引收录进去，就是：是否支持搜索
3. store:用户数据默认存于\_source中，也可以单独存储
4. analyzer

type补充

1. String 又包含：text keyword
2. byte，short，integer，long
3. float, half\_float, scaled\_float，double
4. integer\_range， long\_range， float\_range，double\_range，date\_range
5. Binary Geo\-point、Geo\-shape、IP、Join 等

> 这支持的类型还真挺多...

创建一个索引类型,给my\_first\_index添加一个\<产品\>\<表\>

7.0以前是：http://59.110.167.206:9200/my\_first\_index/\_mapping/product

7.0以后是：http://59.110.167.206:9200/my\_first\_index/\_doc/product

```
curl -XPUT "http://59.110.167.206:9200/my_first_index/_doc/product" -H 'Content-Type: application/json' -d '
{
  "properties": {
    "title": {
      "type": "text",
      "index":true
    },
    "subtitle": {
      "type": "text"
    },
    "price": {
      "type": "float"
    },
    "isonline": {
      "type": "boolean"
    },
    "addtime": {
      "type": "date",
      "format":"yyyy-MM-dd HH:mm:ss||yyyy-MM-dd"
    },
    "stock": {
      "type": "integer"
    }
  }
}'
```

创建一个文档

```
curl -XPUT "http://59.110.167.206:9200/my_first_index/_doc/1" -H 'Content-Type: application/json' -d '
{
    "title": "shouji",
    "subtitle": "zhinengshouji",
    "price": 1200.02,
    "isonline": true,
    "addtime": "2021-06-01 12:00:00",
    "stock": 10
}'
```

## kibana

下载包，解压

同样，官方不推荐直接用ROOT（但也可以用），添加用户组,设置权限

> groupadd kibanagroup;
> 
> 
> useradd kibana \-g kibanagroup \-p kibana
> 
> 
> chown \-R kibana:kibanagroup /soft/kibana

修改个配置参数

> vim /soft/kibana/config/node.options
> 
> 
> \-\-max\-old\-space\-size=256

配置文件

> vim /soft/kibana/config/kibana.yml

```
server.port: 5601  
server.host: "0.0.0.0"  
elasticsearch.hosts: ["http://localhost:9200"] 
server.maxPayloadBytes: 10485760 #这里加了一个0，放大10倍
#logging.dest: logging.dest: /data/logs/kibana/start.log
```

启动

> bin/kibana \-\-allow\-root

看看端口号

> losf \-i:5601

看下api

> http://192.168.1.31:5601/status

这里注意下，kibana启动后会在ES里自动创建了3个索引：

1. .kibana\-event\-log\-7.10.0\-000001
2. .kibana\_task\_manager\_1
3. .kibana\_1

## filebeat

还是elastic 出品，golang编写，直接下载解压

配置示例文件：filebeat.reference.yml（包含所有未过时的配置项）

正常加载的配置文件：filebeat.yml

详情配置说明请跳转

查看下刚刚修改的配置文件语法：

> ./filebeat test config

## 启动

> filebeat \-e \-c filebeat.yml
> 
> 
> ./filebeat \-e \-c filebeat.yml \-d "\*"

filebeat自动创建的索引可能是只读权限，改一下：

```

curl -XPUT -H 'Content-Type: application/json' http://59.110.167.206:9200/_settings -d '
{
    "index": {
        "blocks": {
            "read_only_allow_delete": "false"
        }
    }
}'

curl -XPUT -H 'Content-Type: application/json' http://59.110.167.206:9200/filebeat-7.13.1-2021.06.08-000001/_settings -d '
{
    "index": {
        "blocks": {
            "read_only_allow_delete": "false"
        }
    }
}'
```

查看下已开启的modules:

> ./filebeat modules list

这东西会造成 自动创建索引的时候，共计5000来个mapping...

```
./filebeat modules disable aws

```

## 原理

Filebeat涉及两个组件：查找器prospector（探测者）和采集器harvester（收割者），来读取文件\(tail file\)并将事件数据发送到指定的输出。

## harvester（收割者）

读取单个文件的内容（逐行），并将内容发送到the output，每个文件启动一个harvester, harvester负责打开和关闭文件，这意味着在运行时文件描述符保持打开状态。

## prospector（探测器）

管理 harvester 并找到所有要读取的文件来源

每设置一个日志监听路径 ，即会有一个 prospector

启动成功后，用kibina

> kibana\-\>index management\-\>ck\-\*

查看filebeat在es中新建的相关索引文件，如果为空的话，注意：

1. input 的pash 是否正确
2. 目录下的日志文件是否存在且有内容

日志查看，需要得创建一个 搜索器

kibana\-\>index patterns\-\>

> ck\-local\-\*

error : Payload Too Large

> 修改 kibana.yml 中的server.maxPayloadBytes
