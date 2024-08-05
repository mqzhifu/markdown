# filebeat

## 安装

还是elastic 出品，直接下载解压

配置示例文件：filebeat.reference.yml（包含所有未过时的配置项）

正常加载的配置文件：filebeat.yml

详情配置说明请跳转

## 启动

> filebeat \-e \-c filebeat.yml
> 
> 
> ./filebeat \-e \-c filebeat.yml \-d "publish"

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

这东西会造成 自动创建索引的时候，共计5000来个mapping，关关关...

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
