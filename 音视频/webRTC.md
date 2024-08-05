# webRTC

## 历史

#### GIPS

Global IP Solutions ，公司创立于1999年，专注于音频流业务。2010年5月，google 收购了

> 其也算是 VoIP 领域的公司，使用C\+\+开发

2003 年 skype 使用了 GIPS ，QQ早期也是用GIPS，还有 webEx

#### On2

1995 年创立，一家视频压缩公司。2009 年 8月 被google. 收购，其主要的产品就是：VP8/VP9

#### webRtc

2011 年 6 月，google 推出 webRtc 并开源。

其底层就是基于：

1. GIPS 音频收集、音频传输
2. ON2 的视频解码技术
3. 开源 音频编码技术 opus
4. 最后把协议层打通。

> 音视频技术的版权都有了，基本就可以放弃 adebe 公司了。

#### 小结

其实，一切技术的背后都是生意。都是大资本的角逐，还有大公司的博弈。微软收了 skype \+ msn ，adobe 因为 flash 在 RTC领域长时间处于霸者地位。而 google 在这些领域基本没啥作为，或者说 google 的崛起略晚于传统企业 ，那后面它也通过收购 在补短板。小公司更多的是在一些垂直领域发力，如：音视频编码、3A算法、音视频传输协议等，最终被大公司收购 并整合。

## 概览

Web Real Time Communication

> 看名字就能猜出，最初 GOOGLE 是准备给自家浏览器上使用的。不过后续也出了些 C\+\+ 类库，客户端 手机端 也能兼容一部分。

2017 年 11 ，WEBRTC 加入到W3C，算是给了个正名吧。

> 个人觉得，说到底还是各大公司的博弈。尤其在 flash 没落后，google 反倒是有一统 RTC 的机会。它最大优点：开源！！！

webRTC 虽然也跟 RTMP/HSL RTSP 略像，是协议。不过，个人感觉它更大，更宏观，功能也更多，并不仅仅是个协议，这主要包含几大块：

1. API 调用层
    1. js
    2. libWebRtc
2. session
3. Voice Engine
    1. iSAC / iLBC Codec
    2. NetEQ for voice
    3. 3A算法
4. Video Engine
    1. VP8 Codec
    2. Video jitter buffer
    3. image enhancements
5. Transport
    1. SRTP
    2. Mutilplexing
    3. P2P / STUN \+ ICE \+ TURN
6. 硬件
    1. Audio capture /render
    2. Video capture
    3. Network IO

## API

JS模式，直接写JS代码即可，这个挺方便的，浏览器写代码各种兼容性、性能问题，全自研太累

libWebRtc 就是偏向 native 模式了，客户端程序都可以使用，如：WIN MAC ,andriod ios

## 会话层/信令

webRtc 并未提供该层的实现代码，甚至连详细的规范都没有，估计可能是给开发者更大的自由度吧，其目前主要实现的方式 是：web socket \+ json/SDP 方式。

SDP :session descriptiojn protocol , webRTC 使用此协议做基础信息交互\(协商\)。

它的传输协议格式，KV形式，跟HTTP\-HEADER 有点像：

> \<type\>=\<value\>

WEBRTC 对于类似通信，大致形容成：offer 和 answer ，一般来说 推流的属于 offer ，拉流的属于 answer ，感觉也没太大区别。

它大概是分成了两个大层：

1. session level，用户基础信息：用户名、IP:PORT、session\-id、session\-name
2. media level：
    1. 端口号
    2. 音视频编解码\-格式\(vp8/vp9 h264/h265\)
    3. 音视频\-码率，采样率、通道数，图像帧大小、每秒多少帧、每帧播放多少数据等等
    4. 传输协议
        1. RTP/SRTP/RTCP
        2. 协议通信的具体参数定义
3. ICE Candidate\(网络协商\)
    1. 收集 Candidate
    2. 交换 Candidate
    > 这里 Candidate 中包含的本机IP:PORT，是从ICE\-STUN 获取到的
4. 房间信息
    1. 基础管理，创建/删除
    2. 某用户加入房间、退出房间、掉线等事件

媒体协商的作用：就是让双方找到共同支持的媒体能力，从而能实现彼此之间的音视频通信。

> 比如：A支持 VP8/VP9/H.264/H.265 ,B却只能支持 VP9，那最终协商使用VP9

## Transport

#### ICE

一个框架。能够动态的发现最优的传输路径（不是一种协议），整合了 STUN 和 TURN 两种协议（用于 NAT 穿透）的框架。

1. 通过 STUN 服务收集 NAT 外网地址
2. 通过 TURN 收集中继地址。

Candidate:候选人，当某一端想要直连对端的时候，得知道对端的 IP:PORT、协议信息集

SDP层交换完基础信息，要建立连接的时候，两端初始化基础对象后，会给 STUN 服务器发消息：

1. 收集 Candidate
2. 交换 Candidate
3. 按优先级尝试连接

> 最终将信息是要转发给 信令服务的，由信令服务统一中转消息给到对端

ICE也可以看做是网络协商

#### STUN

session travesal utilities ,为 NAT 后面的客户端机器找出自己的公网IP地址。是一个 server

> 服务端\-端口号 3478，先尝试UDP，失败，就切换TCP

#### TURN

中继器。STUN 使用 NAT 穿透失败。那就只能在S端有一个服务器用于音视频流转发了。

#### 小结

p2p:虽然可以节省带宽资源。但是，如果公司想用这些数据，如：对数据对挖掘、转码/合流，就很麻烦。

另外：民用的，像微信个人视频这些还好，如果是商用的p2p在中国是违法的，得监管。

具说：商用的，大分部不是P2P。除非是1对1 会议且对延迟容忍非常低的

## coturn

实现了：ICE框架 的一个开源软件，官网地址：

> https://github.com/coturn/coturn

ubuntu 下安装

> apt install coturn

这货安装完后，就自动启动了，得先停止一下

> systemctl stop coturn

#### 前置条件1，先申请2个域名：

> turn.seedreality.com
> 
> 
> stun.seedreality.com

给域名添加 HTTPS ssl 证书

> 如果仅仅是测试一下 coturn 这两个域名不申请也行，用IP也能访问。但要连起来测试，或者正式使用不行，必须得有域名，因为：浏览器要HTTPS

#### 前置条件2，打开服务器端口号：

默认端口\-不加密：3478

默认端口\-已加密：5349

阿里云安全组/防火境况 端口配置：

> 3478/5349 两个都得支持：TCP\+UDP
> 
> 
> 40000/41000 TCP\+UDP 要配置，出口：TCP\+UDP 也要配置，一共是4个

#### 前置条件3，生成 SSL：

openssl req \-x509 \-newkey rsa:2048 \-keyout /data/turn\_server\_pkey.pem \-out /data/turn\_server\_cert.pem \-days 99999 \-nodes

#### 配置文件

> vim /etc/turnserver.conf

```
#这两变量我是没看懂嘛意思，必须得是域名。好像是OAUTH验证时使用
server-name=xiaoz_webRtc
realm=coturn.seedreality.com

#注：这里如果是阿里云 要监听内网
listening-ip=0.0.0.0
external-ip=8.142.177.235
#注：这里如果是阿里云 要监听内网
relay-ip=8.142.177.235

#这里没敢开太大的区间，怕阿里云限制
min-port=40000
max-port=41000

fingerprint
log-file=/home/ubuntu/server/turn/turnserver.log
verbose

#正常用户名密码
user=xiaoz:123456
#这个我没找见是哪里使用
cli-password=11223344

#这里是直接从阿里下载NGINX格式的密钥
#可能是给 coturn.seedreality.com 用，也可能是给 web-admin，这里不确定
pkey=/data/turn_server_pkey.pem
cert=/data/turn_server_cert.pem

#web-admin 会使用 ，如果要使用还得安装 SQLlite
#userdb=/var/lib/turn/turndb

```

#### 启动

> turnserver /etc/turnserver.conf

#### 测试

用工具测试，地址如下：

> https://webrtc.github.io/samples/src/content/peerconnection/trickle\-ice/

操作测试工具：

1. 填写服务端信息：

```
turn:8.142.177.235:3478
xiaoz
123456
```

> 如果有申请域名也可以这样写 turn:turn.seedreality.com:3478

1. 点击：add server
2. 点击：gather candidates

返回结果集分析：

|选修项\-类型|英文名|如何传给对象    |用法                                                                                  |
|------------|------|----------------|--------------------------------------------------------------------------------------|
|主机        |host  |信令服务器      |从网卡中获取的本地传输地址 ，如果此地址位于NAT之后，则为内网地址                      |
|服务器反射  |srflx |信令服务器      |从发送给STUN服务器的 binding 检查中获取的传输地址。如果此地址们于NAT之后，则为内网地址|
|反射        |prflx |stun binding请求|从对端发送的stun binding 请求获取的传输地址 。这是一种在连接检查期间新发生的候选项    |
|中继候      |relay |信令服务器      |媒体中继服务器的传输地址 。通过 使用TURN ALLOCATE 请求获取                            |

> 这里主要关注的是： relay 返回的外网IP是否正确，还有：如果出现 done,大概率是 OK

#### coturn 安装/编译完后，有几个可执行命令:

1. turnadmin：用来管理账户
2. turnserver：服务器
3. turnutils\_stunclient 用于测试stun服务
4. turnutils\_uclient 用于测试turn服务. 模拟多个UDP,TCP,TLS or DTLS 类型的客户端

> 测试：turnutils\_uclient \-u USERNAME \-w PASSWORD \-p PORT \-v LISTEN\-ADDRESS

#### 小结

这个东西好复杂，主要都是偏向网络层的配置，对这块不熟悉，搞起来挺费时间的。

而且，一些配置信息依然不懂，如：

1. relem 与 server\-name
2. 它的证书配置项到底是给谁用的？
3. cli 的配置块 如何用？
4. web\-admin 最终因为 https 没弄好，也没试上

## APP\-RTC

GOOGLE实现的一个 WEBRTC 代码库

## collider

信令

## jitterBuffer

抖动缓冲区，它是源于语音领域的一个算法吧。但也可以放大到视频层面。核心还是它的算法思路比较适合音视频传输的领域

其主要功能就是：

1. 对网络传输数据的质量、稳定\(Qos\)做出的保证
2. 对音视频的数据流做出些优化
3. 平滑的向解码模块输出数据包/帧，尽量不卡顿

> 它是在 RTP包 和 解码 之间，接收已拆的RTP包，加工处理后，解码器从它这里再拉取处理好的数据

有时候一帧的数据，可能大于一个 MTU，那么，就得拆分成两个包传输。而网络传输是有很大不确定因素的，有时候一帧被拆成了4个包，正常来看 1 2 3 4 ，接收也是 1 2 3 4 ，但实际情况可能 是 4 1 2 3，jitterBuffer 就把这一帧数据的包调整成： 1 2 3 4 ，然后合成一帧数据。

> 这里也可能丢包，它也会做出相应的处理。如：视频帧，I帧 P帧；根据 GOP 再做一次调整策略

最后，解码器 取的数据，基本上都是正确的，直接解文件、解码，播放/渲染即可

具体功能列表：

1. 丢包重传、乱序\(排序\)
2. 组帧、判断帧完整性、判断帧可解码性、
3. 关键帧请求
4. 计算延迟时间

根据抖动来动态调整 buffer 长度的过程:

1. 网络差时：增加缓存时间
2. 网络好时：减少缓存时间

难点：准确计算出 延迟值

用时间换取流畅，也可以间接理解为：保证音视频不卡顿

这里有个问题，是不是只有RTP才用 jitterBuffer ，因为 RTP偏向使用UDP，所以 重传 乱序 需要 jitterBuffer 处理？

## NetEQ

netEQ = jitter buffer \+ decoder \+ PCM信号处理

## 总结

WEBRTC 更像是一个大的框架，而不单单是某一个传输协议。并且它还有一些裸流的处理，如：降噪、降回声，滤镜等。还有如信令的各种协商，NAT穿越，对摄像头、麦克风的OS\-API的调用等等吧。

GOOGLE 应该是想大一统 RTC 领域，接班曾经的 adobe flash，但其它大厂肯定是不干的，这也导致了：

1. webRtc 其实比较全面了，但是各大公司并没有完全使用WEBRTC，而是配合原 adebo 的RTMP ，苹果的 HSL 等一起使用
2. 各大厂也有自己的RTC协议，间接导致程序员需要各种兼容

但不管怎么说吧，它能解决浏览器端的兼容性，同时又能有较好的延迟，至少：程序员们不用再为浏览器的性能低、兼容差而费神了\(不需要完全自写JS代码\)。
