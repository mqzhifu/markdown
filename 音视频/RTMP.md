# RTMP

## 概览

Real Time Messaging Protocol:实时消息协议，基于 TCP，默认使用端口1935

早期由 Macromedia 公司创建，后期被 adobe 公司收购了。随后， adobe 公司开源了 一部分代码。

它是应用层的协议，用来解决多媒体数据传输流的多路复用（Multiplexing）和分包（Packetizing）的问题。主要是服务于 flash player的。

> 具说2012就已经出现了,挺前卫的
> 
> 
> 2017年 ，adebo 宣布3年后将放弃 flash 播放器。2020年12月31日 正式宣布 放弃。连带着它的自有协议也无人维护了:RTMP/HTTP\-FLV

直播拉流的一个URL：

> rtmp://www.baidu.com/record/e10adc3949ba59abbe56e057f20f883e

## 传输流程

\!. C端，摄像头采集到的视频流\(数据\)，拆成若干message

2. 将message拆分成若干的 chunk

3. 发送 chunk

4. S端也类似

## 协议体概览

![Handshake-done.png](image/Handshake-done.png)

1. 握手
    1. 协议倒是简单，不过一共6次网络IO，好像C0 C1 可以合并成一次，S0 S1 S2可以合并成一次，最终是3次。
    2. C1:时间戳\(4字节\)\+零值\(4字节\)\+随机数据\(1528\)=1536个字节
    > 1536 这个受TCP MSS，会被拆成两个包，不知道1536这值怎么来的，随机数是用来互相确认的
    1. C2/S2:时间戳1\(4字节\)\+时间戳2\(4字节\)\+随机数据\(1528\)=1536个字节
    > 零值没了，变成了时间戳2，可确认对端的本地时间
2. 连接

> fmt=2 ,Message Header 中的 message type=17/20 ,17=AMF3

1. 传输数据

协议体的格式：

![RTMP-TOTAL.png](image/RTMP-TOTAL.png)

基础数据头（Basic Header）：保存 CS ID、Chunk Type（决定 Msg Header 类型）

消息数据头（Message Header）：包含被发送消息的相关信息，类型Chunk Type决定

扩展时间戳（Extended Timestamp）（32\-bits）：消息头携带的时间戳扩展位，主要是配合 Message Header 内的时间戳使用

> 只能ts大于24位，此字段值才会出现

## Basic Header

![rtmpt-basic-header.png](image/rtmpt-basic-header.png)

chunk stream ID \+ chunk type

1. chunk stream id: 一般被简写为CSID，用来唯一标识一个特定的流通道。也可以理解为：信道ID
2. chunk type ：chunk的类型。也叫：format message type\( fmt\)

#### chunk type

固定占用2个bit。它决定了后面Message Header的格式

#### csid

可能是1/2/3个字节，取决csid具体的数值：

0: Basic Header 总共要占用 2 个字节，CSID 在 \[64,319\] 之间;

1: Basic Header 3 个字节，CSID 在 \[64,65599\] 之间;

2: 代表该 chunk 是控制信息和一些命令信息。

2之后的数字可随意定制，当然最好不要太长.adebe公司貌似给了一套标准：

3：命令信道

5 数据流信道

6 视频

7 音频

Basic Header:它为什么是变长的？定长不是更好？因为：它其实是通过CS ID 实现的多路复用，那么必然就得在通信的时候增加些字节来控制这些。只能压榨Basic Header的长度了

#### 协议控制消息（Protocol Control Message）

当：csid=2，代表协议控制消息。那么在 Message Header 中message id 就必须 = 0.

连带的，消息体中的 Message Header 中的 message type值就开始起作用了：

1. Type\_ID=1,Set Chunk Size：设置chunk中Data字段所能承载的最大字节数，默认为128B，通信过程中可以通过发送该消息来设置chunk Size的大小（不得小于128B），而且通信双方会各自维护一个chunkSize，两端的chunkSize是独立的。
2. Type\_ID=2,Abort Message ：当一个Message被切分为多个chunk，接受端只接收到了部分chunk时，发送该控制消息表示发送端不再传输同Message的chunk，接受端接收到这个消息后要丢弃这些不完整的chunk。Data数据中只需要一个CSID，表示丢弃该CSID的所有已接收到的chunk。
3. Type\_ID=3,Acknowledgement ：当收到对端的消息大小等于窗口大小（Window Size）时接受端要回馈一个ACK给发送端告知对方可以继续发送数据。窗口大小就是指收到接受端返回的ACK前最多可以发送的字节数量，返回的ACK中会带有从发送上一个ACK后接收到的字节数。
4. Type\_ID=4,User Control Message 用户控制消息
5. Type\_ID=5,Window Acknowledgement Size ：发送端在接收到接受端返回的两个ACK间最多可以发送的字节数。
6. Type\_ID=6,Set Peer Bandwidth：限制对端的输出带宽。接受端接收到该消息后会通过设置消息中的Window ACK Size来限制已发送但未接受到反馈的消息的大小来限制发送端的发送带宽。如果消息中的Window ACK Size与上一次发送给发送端的size不同的话要回馈一个Window Acknowledgement Size的控制消息。

> 以上1~6主要是给协议本身使用

1. Type\_ID=8,RTMP 音频数据包
2. Type\_ID=9,RTMP 视频数据包
3. Command Message\(命令消息，Message Type ID＝17或20\)：表示在客户端和服务器间传递的在对端执行某些操作的命令消息，如connect表示连接对端，对端如果同意连接的话会记录发送端信息并返回连接成功消息，publish表示开始向对方推流，接受端接到命令后准备好接受对端发送的流信息，后面会对比较常见的Command Message具体介绍。当信息使用AMF0编码时，Message Type ID＝20，AMF3编码时Message Type ID＝17。
4. Data Message（数据消息，Message Type ID＝15或18）：传递一些元数据（MetaData，比如视频名，分辨率等等）或者用户自定义的一些消息。当信息使用AMF0编码时，Message Type ID＝18，AMF3编码时Message Type ID＝15。
5. Shared Object Message\(共享消息，Message Type ID＝16或19\)：表示一个Flash类型的对象，由键值对的集合组成，用于多客户端，多实例时使用。当信息使用AMF0编码时，Message Type ID＝19，AMF3编码时Message Type ID＝16。
6. Audio Message（音频信息，Message Type ID＝8）：音频数据。
7. Video Message（视频信息，Message Type ID＝9）：视频数据。
8. Aggregate Message \(聚集信息，Message Type ID＝22\)：多个RTMP子消息的集合 。
9. User Control Message Events\(用户控制消息，Message Type ID=4\):告知对方执行该信息中包含的用户控制事件，比如Stream Begin事件告知对方流信息开始传输。和前面提到的协议控制信息（Protocol Control Message）不同，这是在RTMP协议层的，而不是在RTMP chunk流协议层的，这个很容易弄混。该信息在chunk流中发送时，Message Stream ID=0,Chunk Stream Id=2,Message Type Id=4。

> 1~6 被保留，供协议自己使用

![ppp.png](image/ppp.png)

#### 分析

chunk type 占2位，剩下6位，那就是2^6=64\-1 =63, 最大：0~63 ，再减掉特殊标识： 0/2/3 = 3~63

最终：如果是一个字节的情节下，cs id可使用的值为：3~63

> 如果一条普通消息\(非控制类消息\) csid 在：3~63范围内，Basic Header 一个字节就够了

首字节的6位是：000002 ，这个是控制信息 Protocol Control Messages

## Message Header

![rtmp-msg-header.png](image/rtmp-msg-header.png)

> cont = continue

它的格式/长度受：Basic Header 中的 chunk type\(fmt\) 影响

> fmt: 0 1 2 3

1. fmt=0:共11个字节，如图
    1. timestamp\(3\-Byte\) 当前消息时间戳: 有效位 24 bits，如果超出16777215（0xFFFFFF）则启用扩展时间戳（Extended Timestamp）。
    > 扩展位启用时，timestamp 位恒定为 16777215，通过还原 32 bits 的扩展位，加合为有效时间戳数据。时间戳在运用上对于不同消息类型会有区分，type 0 时为绝对时间戳，type 1/2 时为相对时间戳（时间差值）。
    1. message length\(1\-Byte\)：消息头长度，携带 Chunk Header 数据长度信息（单位：Byte）
    2. message length \(cont\) \(2\-Byte\):消息体长度,携带 Chunk Data 数据长度信息（单位：Byte）
    3. message type 消息类型\(1\-Byte\)：携带消息类型信息，这是实际消息的类型，区别于消息头。
    4. message stream id 字段\(1\-Byte\)：消息归属消息流 ID 标志位，指定当前消息所属信道分类
    5. message stream id \(cont\) \(3\-Byte\)：消息内容对应数据流 ID 标志位，指定数据所属数据流信道

chunk type=1:Message Header占用7个字节，省去了表示msg stream id和message stream id \(cont\)的4个字节

表示：此chunk和上一次发的chunk所在的流相同，如果在发送端只和对端有一个流链接的时候可以尽量去采取这种格式。

chunk type=2：Message Header占用3个字节，相对于type＝1格式又省去了表示消息长度的3个字节和表示消息类型的1个字节，表示此chunk和上一次发送的chunk所在的流、消息的长度和消息的类型都相同。余下的这三个字节表示timestamp delta，使用同type＝1 。

chunk type=3:1个字节，空数据，用于处理延迟消息

#### fmt: 0 1 2 3 使用场景

0：通信建立开启时 或切换到后台

1：视频流数据

2：音频流数据

3：空数据标准，时差标准，延迟标记

## Chunk /data\(Body\)

AMF:Action Message Format:类似JSON，XML的二进制数据序列化格式

不同的消息类型\(message type\)，对应不同的数据体

![amf.png](image/amf.png)

## 结合分析

csid \+ messageid = 多路由复用

> A视频会议ID \+ 音频ID
> 
> 
> A视频会议ID \+ 视频ID
> 
> 
> 就是一个完整的 音视频流数据了

视频流的数据因为很大很多，如果使用传统TCP\+RTC 模式，一个TCP连接瞬间被打进去很多数据，肯定会阻塞，如果在这期间有些其它的小包优先集高，但迟迟发不出去的情况




# 总结

1. 把大的视频流拆成若干的 chunk data
2. 加上cs id 多路复用提高性能
3. 把流拆分成：音频+视频
3. 每个流加上属性：时间（视频在哪个时间段上）