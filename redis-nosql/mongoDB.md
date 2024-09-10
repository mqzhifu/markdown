
# 基础

C++ 编写，基于 document(bson) 模式。
mongdoBB：以 json 为数据格式管理，实现了分布式存储 的 数据库系统

底层核心：
- 分布式：原生支持，对比 DBMS 很简单。
- BSON：数据的存储格式，支持 NOSQL 的所有属性
- 索引：它支持格式的更多，更丰富

MongoDB 对比 DBMS:

| MongoDB            | MySQL     |
| ------------------ | --------- |
| DB                 | DB        |
| collection         | 表         |
| 文档                 | 行         |
| 字段                 | 列         |
| 主键                 | 主键        |
| join               | 嵌套文档      |
| 地理位置索引 hash索引 全文索引 | 普通索引 唯一索引 |


>MYSQL 的全文索引 仅限 myisam 且性能不高
一行数据就是一个：document


整体构架：基础层 +  存储引擎
> 跟 mysql 差不多

基础层负责：
-  sql query
- 网络连接

存储引擎：数据的具体存储方式

# BSON

类似 json 格式，简单区别：
- 二进制形式的存储格式
- 支持内嵌的文档对象和数组对象
- 支持更多数据类型：decimal128 date byte array 正则




```
{
    title:"MongoDB",
    last_editor:"192.168.1.122",
    last_modified:new Date("27/06/2011"),
    body:"MongoDB introduction",
    categories:["Database","NoSQL","BSON"],
    revieved:false
}
```



```
{
    name:"lemo",
    age:"12",
    address:{
        city:"suzhou",
        country:"china",
        code:215000
    } ,
    scores:[
        {"name":"english","grade:3.0},
        {"name":"chinese","grade:2.0}
    ]
}
```


优点：
- 数据类型更多
- 性能好：遍历快

缺点：占用空间大


底层存储的结构：

```
{"hello": "world"}
```

```
\x16\x00\x00\x00 
\x02 
hello\x00 
\x06\x00\x00\x00world\x00 
\x00
```

每一行分析：
- `\x16\x00\x00\x00`：
	- x 表示16进制的方式。
	- -4个字节表示文档的大小，包括文档末尾的'\0','\0'是\x00 0x16十进制是22，这个文档的大小是22个字节 采用小端（Little Endian) 
- `\x02`：一个字节表示 value 的类型是string，字符串编码使用的是UTF-8
- `hello\x00`：表示以'\0'结尾的字符串
- `\x06\x00\x00\x00world\x00`:前4个字节表示字符串world的长度
- `\x00` 结束符

再看一个例子：

```
{"BSON": ["awesome", 5.05, 1986]}
```

数组：['red', 'blue'] 
- 将要编码为{'0': 'red', '1': 'blue'}
- key必须按照数值大小递增排序(升序)。 
 
也就是：["awesome", 5.05, 1986] 将被编码为
```
{ "0":"awesome", "1":5.05，"2":1986} {"BSON": [ "0":"awesome", "1":5.05，"2":1986]}
```


对应是底层存储结构：

```
\x31\x00\x00\x00
\x04BSON\x00
\x26\x00\x00\x00
\x02\x30\x00\x08\x00\x00\x00awesome\x00
\x01\x31\x00\x33\x33\x33\x33\x33\x33\x14\x40
\x10\x32\x00\xc2\x07\x00\x00
\x00
\x00
```


解释每一行的意思 

- `\x31\x00\x00\x00`： 4个字节表示文档的大小，x31的10进制是49，这个文档的大小是49个字节
- `\x04BSON\x00`： \x04表示 value 的类型是数组
- `\x26\x00\x00\x00` ： 4个字节表示数组的大小即中括号的内容，x26的10进制是38
- `\x02\x30\x00\x08\x00\x00\x00awesome\x00`： 
	- \x02 表示 value 的类型是 string 
	- x30表示 key，字符0的ASCII码是48,16进制是x30 纵向看正好是x30，x31，x32 
	- \x08\x00\x00\x00 4个字节表示：awesome\x00 的长度
- `\x01\x31\x00\x33\x33\x33\x33\x33\x33\x14\x40`：
	- \x01 表示 64 位的二进制浮点数 
	- x31\x00表示以'\0'结尾的字符串1,字符1的ASCII码是x31 x33\x33\x33\x33\x33\x33\x14\x40 double的5.5转换成16进制为40 14 33 33 33 33 33 33

`\x10\x32\x00\xc2\x07\x00\x00`：\x10表示32位的整数。\x32\x00表示以'\0'结尾的字符串2,字符2的ASCII码是x32 \xc2\x07\x00\x00也就是16进制的7c2转换成10进制是1986

最后两行的`\x00 \x00`：结束符。


从二进制存储结构更能清晰的看出 BSON 到底是什么~
它是对 json格式 封装了一层，增加了：数据类型、数据的总长度、将字符流转成二进制流、数组有顺序性。


# 分布式 / 集群

主从复制（Master-Slaver）、副本集（Replica Set）和分片（Sharding）模式。

Master-Slaver： 主从副本的模式
>目前已经不推荐使用，新版本弃用了

Replica Set ：一主多从。主节点用于写，从节点可以读。数据被复制多份保存，不同服务器保存同一份数据，在出现故障时自动切换，实现故障转移。
>算是对 主从复制 的升级

Sharding ：适合处理大量数据，它将数据分开存储，不同服务器保存不同的数据，所有服务器数据的总和即为整个数据集。


#### Sharding / 分片 


优点：
- 海量数据后，主从的复制模式，单机不可能存储所有数据，太大了
- 读写请求性能更高。被分散到好多的机器上去执行了


MongoDB Sharded cluster 支持 2 种分片方式：
- 范围分片，通常能很好的支持基于 shard key 的范围查询
- Hash 分片，通常能将写入均衡分布到各个 shard


# 字段

当向一个集合插入一条数据时，mongoDb会默认增加一列：\_id 
`_id`：ObjectId 类型，主键唯一索引（12个字节，十六进制）

4字节：时间戳
3字节：机器码
2字节：进程ID/PID
3字节：计数器

>因为ID字段中饮食 创建时间，所以不需要再单独创建字段了

| 数据类型               | 描述                                                          |
| ------------------ | ----------------------------------------------------------- |
| String             | 字符串。存储数据常用的数据类型。在 MongoDB 中，UTF-8 编码的字符串才是合法的。              |
| Integer            | 整型数值。用于存储数值。根据你所采用的服务器，可分为 32 位或 64 位。                      |
| Boolean            | 布尔值。用于存储布尔值（真/假）。                                           |
| Double             | 双精度浮点值。用于存储浮点值。                                             |
| Min/Max keys       | 将一个值与 BSON（二进制的 JSON）元素的最低值和最高值相对比。                         |
| Array              | 用于将数组或列表或多个值存储为一个键。                                         |
| Timestamp          | 时间戳。记录文档修改或添加的具体时间。                                         |
| Object             | 用于内嵌文档。                                                     |
| Null               | 用于创建空值。                                                     |
| Symbol             | 符号。该数据类型基本上等同于字符串类型，但不同的是，它一般用于采用特殊符号类型的语言。                 |
| Date               | 日期时间。用 UNIX 时间格式来存储当前日期或时间。你可以指定自己的日期时间：创建 Date 对象，传入年月日信息。 |
| Object ID          | 对象 ID。用于创建文档的 ID。                                           |
| Binary Data        | 二进制数据。用于存储二进制数据。                                            |
| Code               | 代码类型。用于在文档中存储 JavaScript 代码。                                |
| Regular expression | 正则表达式类型。用于存储正则表达式。                                          |

use 不存在的DB，会自动创建一个DB

# 搜索

它支持：正则查询，如：以A开头的字段，包含某字段的字段
如果字段是数组，可对数组进行操作


# 索引

单字段索引：基于单个字段的索引。
复合索引：基于多个字段组合的索引。
文本索引：用于支持全文搜索。
地理空间索引：用于地理空间数据的查询。
哈希索引：用于对字段值进行哈希处理的索引。


```
// 创建唯一索引  
db.collection.createIndex( { field: 1 }, { unique: true } )  
  
// 创建后台运行的索引  
db.collection.createIndex( { field: 1 }, { background: true } )  
  
// 创建稀疏索引  
db.collection.createIndex( { field: 1 }, { sparse: true } )  
  
// 创建文本索引并指定权重  
db.collection.createIndex( { field: "text" }, { weights: { field: 10 } } )  
创建地理空间索引  
对于存储地理位置数据的字段，可以使用 2dsphere 或 2d 索引类型来创建地理空间索引。  
  
// 2dsphere 索引，适用于球形地理数据  
db.collection.createIndex( { location: "2dsphere" } )  
  
// 2d 索引，适用于平面地理数据  
db.collection.createIndex( { location: "2d" } )
```
# 实际应用场景 

# 存储引擎
