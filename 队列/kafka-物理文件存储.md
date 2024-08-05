# 消息存储的物理结构


#### 分区文件夹

每一个 partition ，在服务器上都会有一个文件夹
>文件夹名格式：主题名+分区号

包含如下：

|  |  |  |
|:---|:---|:---|
| 消息 | 00000000000000000010.log  | 有若干个，也叫 segment |
| offset 索引 | 00000000000000000010.index | 消息的物理地址的偏移量索引文件 |
| 快照 | 00000000000000000010.snapshot | 对幂等型或者事务型producer所生成的快照文件 |
| time 索引 | 00000000000000000010.timeindex  | 映射时间戳和相对offset的时间戳索引文件 |
| 选举相关 | leader-epoch-checkpoint | 每一任leader开始写入消息时的offset, 会定时更新   |  
文件切割方式：大小、时间(可以在 server.properties 配置)

segment 组：消息、offset 索引、time 索引、快照
>这4个文件都是成组出现，不会单独出现一个
>文件被切割后，会有若干个这样的组

文件内容的存储格式为：二进制(得用特定工具查看内容)
>leader-epoch-checkpoint 可以用 vim 直接查看


#### 消息文件的具体存储内容

指令行-工具查看文件内容：
>/opt/kafka/bin/kafka-run-class.sh kafka.tools.DumpLogSegments --files  00000000000000000010.log  --print-data-log

```
Starting offset: 10

baseOffset: 14 lastOffset: 15 count: 2 baseSequence: -1 lastSequence: -1 producerId: -1 producerEpoch: -1 partitionLeaderEpoch: 0 isTransactional: false isControl: false position: 296 CreateTime: 1712936404265 size: 85 magic: 2 compresscodec: NONE crc: 182335447 isvalid: true

| offset: 14 isValid: true crc: null keySize: -1 valueSize: 5 CreateTime: 1712936403314 baseOffset: 14 lastOffset: 15 baseSequence: -1 lastSequence: -1 producerEpoch: -1 partitionLeaderEpoch: 0 batchSize: 85 magic: 2 compressType: NONE position: 296 sequence: -1 headerKeys: [] payload: xiaoz

| offset: 15 isValid: true crc: null keySize: -1 valueSize: 4 CreateTime: 1712936404265 baseOffset: 14 lastOffset: 15 baseSequence: -1 lastSequence: -1 producerEpoch: -1 partitionLeaderEpoch: 0 batchSize: 85 magic: 2 compressType: NONE position: 296 sequence: -1 headerKeys: [] payload: haha

```

具体字段：

|  |  |  |
|:---|:---|:---|
| baseOffset | 批次（一次可能发送多条消息）初始偏移量 |  |
| lastOffset | 批次（一次可能发送多条消息）最后偏移量 |  |
| count | 批次-发送消息数量（lastOffset - baseOffset + 1 = count） |  |
| lastSequence |  |  |
| producerId | 生产者的编号，用于支持幂等性 |  |
| producerEpoch |  |  |
| partitionLeaderEpoch |  |  |
| isTransactional |  |  |
| isControl |  |  |
| position |  |  |
| CreateTime | 创建时间，13位,unix 时间戳 |  |
| size | 整条记录的大小 |  |
| magic | 内容格式类型/kafka 协议版本号 |  |
| compresscodec | 压缩工具 |  |
| crc | 校验码 |  |
| isvalid |  |  |
|  |  |  |  

|  |  |  |
| :--- | :--- | :--- |
| offset | 消息ID，类似自增ID，记录ID，一个分区的消息ID |  |
| isValid |  |  |
| crc |  |  |
| keySize |  |  |
| valueSize |  |  |
| CreateTime | 创建时间，13位,unix 时间戳 |  |
| baseOffset | 批次（一次可能发送多条消息）初始偏移量 |  |
| lastOffset | 批次（一次可能发送多条消息）最后偏移量 |  |
| baseSequence |  |  |
| lastSequence |  |  |
| producerEpoch |  |  |
| partitionLeaderEpoch |  |  |
| batchSize |  |  |
| magic | 内容格式类型/kafka 协议版本号 |  |
| compressType |  |  |
| position | 存储该记录的起始位置，position+size=下一条 position 位置 |  |
| sequence |  |  |
| headerKeys |  |  |
| payload | 消息内容 |  |
发送消息，是以批次为单位，可以是一条或多条，所以，消息的存储格式被分成了两大类：
1. 批次汇总，在头部
2. 每一条具体的消息

每一条消息，也可以理解成一条记录，因为除了消息内容，还有18+个字段
>假设我们发一条消息：a ，就一个字符，但到了kafka 存储的时候至少也得18+1 个字符


#### 索引文件的具体存储内容：
>/opt/kafka/bin/kafka-run-class.sh kafka.tools.DumpLogSegments --files  00000000000000000010.index  --print-data-log

```
offset: 387 position: 6094
offset: 518 position: 10734
offset: 622 position: 16367
offset: 1138 position: 27121
offset: 1246 position: 39566
offset: 1359 position: 44315
offset: 1731 position: 49284
offset: 2103 position: 65649      
offset: 2475 position: 82014      
offset: 2847 position: 98379      //下方一批数据：baseOffset: 2476——lastOffset: 2847
offset: 2909 position: 114744
```

索引文件：一列是 offset，一列是 position
>position：这条数据在 Segment 文件中的物理偏移量
>offset：这条数据在这个 Segment 文件中的位置，是这个文件的第几条

#### 时间文件的具体存储内容

```
Dumping 00000000000001778276.timeindex
timestamp: 1645765272680 offset: 1778427
timestamp: 1645765272681 offset: 1778442
timestamp: 1645765275680 offset: 1778507
timestamp: 1645765275681 offset: 1778521
timestamp: 1645765278680 offset: 1778583
timestamp: 1645765278681 offset: 1778597
timestamp: 1645765281680 offset: 1778667
```

#### 小结

- 所有的消息，按照一定格式，被顺序的存储在一个文件中
- 再建立2个索引文件，用于快速查找
- 再建立1个快照文件，用于幂等、事务

感觉有点像是一个数据库了，但它没有DBMS复杂：
- 可以顺序存储
- 任意切割
- 使用简单索引
- 内容压缩


也可以到 zk 中查看
zkCli.sh
