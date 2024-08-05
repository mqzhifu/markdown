
# 概览

AMQP 协议（advanced message queuing protocol）


![[rabbitmq-概览.png]]

![[rabbitmq-整体流程.png]]




| key | value |
|:---|:---|
| Virtual host | 类似namespace，一个实例上，为不同业务划分不同的实例范围  |
| connect | tcp 连接 |
| Channel | 把一个TCP连接，再划分为若干的 channel |
| Exchange | 消息队列交换机 |
| broker |  一台机器上的一个rabbitmq 实例 |
| queue | 队列，实际存储消息的容器 |
| binding | exchang 绑定 queue  或  exchang 绑定 exchang |
|  |  |
|  |  |  
 
# Exchange

消息队列交换机，消息发送到 RabbitMQ 中后，会首先进入一个交换机，然后由交换机负责将数据转发到不同的队列中

#### arguments

| key                 | value                          |     |
| :------------------ | :----------------------------- | --- |
| alternate  exchange | 备份交换器。存储：没有绑定队列或者没有 routkey 匹配 |     |
| auto  delete        | 当所有绑定队列都不在使用时，是否自动删除交换器        |     |
| type                | direct fanout topic header     |     |
| internal            | exchange to exchange           |     |
| durable             | 是持久的，重启  broker 还存在            |     |
| transient           | 是暂存的，重启  broker 没有了            |     |
|                     |                                |     |

#### direct

需要 和 routingKey 配合使用。只要 routingKey = 发送者的 routingKey， queue  就会收到消息
> 默认的类型就是 direct

#### fanout 

广播消息。跟此队列绑定，发消息，即所有队列都能收到该条消息
>不需要 routingKey 

#### topic

必须要配置一个 RountingKey，RountingKey：由字符组成，通过dots（也就是 . ）来进行分割，也可以设置通配符

#### header

根据消息头里的 key 进行匹配路由
x-match”，这个键的 Value 可以是any或者all
# Queue

真正收发消息的容器，需要与 Exchange 进行绑定，才能使用


#### 属性/arguments

| key | value |
| :--- | :--- |
| x-max-length | 队列消息最大数，超过后，末尾消息被删除 |
| x-max-length-byte | 消息内容最大字节 |
| x-message-ttl | 消息存活时间(毫秒) |
| x-max-priority | 消息优先级。可设置最大权重值（正整数） |
| x-expires | queue 存活时间(毫秒)，多久未访问 |
| x-dead-letter-exchange | 投递失败，转到此exchange |
| x-dead-letter-routing-key | 投递失败，根据key  选择exchange |
| type | classic Quorum |
| exclusive | 排它，连接断开后，队列删除 |
| auto  delete | 所有消费者都断开连接了,是否自动删除 |
| durability | durable 持久化 |
| Overflow behaviour | 队列满后，是从头部/尾部删除消息 |


#### Quorum：仲裁队列。(3.9.x以后)是基于Raft一致性协议实现的一种新型的分布式消息队列
>消息需要有集群中多半节点同意确认后，才会写入到队列中

#### Classic：经典队列，早期使用


#### 死信队列/死信 exchange

消息：TTL 失效、队列消息满了，消费者拒绝消费此消息，会进入死信队列。
投递消息： routing key 匹配失败， exchange 设置了死信 exchange 


有些人，把死信队列做成了 延迟队列。比如：把 x-message-ttl 设置的非常低
虽然也能实现延迟这个功能，但感觉不是这么用的，最好是用它官方的延迟插件

>但 RabbitMq 只会检查第一个消息是否过期

# 绑定

exchang binding exchang
exchang binding queue

队列想要使用，必须得它绑定一个 exchange




# 消息回执


消费者拿到一条消息后，需要告知 S 端状态


#### 模式：

1. none：关闭 ACK
2. manual：手动  ACK
3. auto：自动 ACK。没有异常的前提下，自动确认消息

#### 拒绝消息：

>错误的消息或者无法处理的消息

1. nack：拒绝一条消息。可先参数：requeue
2. reject：拒绝多条消息。可先参数：requeue
3. recover：拒绝一条消息，并直接 requeue
>requeue：被拒绝的消息，是否重新放回队列中，如果放回，可能会循环一直读此消息。如果不放回，可以进入死信队列


# 消费者的流控

消费者，拿到消息的速度可以很快，但是具体消耗这条消息，可能是比较耗时的，甚至出现长时间阻塞。
但，队列系统是不停把消息推送过来，如果消费不及时，推送的消息又不断，很容易出问题，最好是能做到一定的流控

#### prefetch_count 

消费者在监听之前，设置此值告知S端，一次可推送最多消息数，如果未能及时ACK，就不要再推送了

prefetch=n：表示一次从队列中获取 n 条消息。其余消息存放在队列中。
prefetch_count : 未进行`ack`的最大消息数量上限


削峰，正常 MQ 收到消费后，消费者会一次拿到全部。如果消费者的服务器负载过高，最好还是分批次给到消费者


# 持久化

持久化，同时开启：Exhange、Queue、Msg。
>具体看， exchange 要开启  durable ， queue 开启 durable 。 queue 要关闭：exclusive auto-delete   x-expire msg-ttl  


# 可靠性

一次消息从发送到消费，经历的过程还是挺多的。保证可靠性就得全链路考虑：

#### 持久化

Exhange、Queue、Msg 都要做持久化
关闭 auto-delete  exclusive  x-expire msg-ttl 
#### 投递

生产者在发送消息时，要采用  publish_confirm 模式，也可以开启事务模式，要接收 S 端的应答

#### 消费者

所有的消息，手动 ack 告知S端消息已处理

#### 3方备份

可以在每次投递的时候，也同时往 redis 里插入一条记录
拒绝一次消费，也要记录一下

#### 备份 

exchang 开启 alter-exchang ，queue 开启死信队列


# 消费者

生产者 -> exchang -> 队列  <-  一个消费者
>这种肯定是满足的

生产者 -> exchang -> 队列  <-  多个消费者
>这种肯定是满足的，也可以看成 多个消费者，负载均衡

生产者 -> exchang -> 队列
生产者 -> exchang -> 队列       一个消费者
>这种不能满足，也就是一个消费者订阅多个 topic


# 集群/负载均衡

这里假设是3台机器

每台机器上启动一个 rabbitMQ 实例，设置 erlang cookie ，3个实例之间可以通信

每台机器都会保存无数据： exchange  queue  元/基础 配置信息

真正的 queue 数据，是分散到每台机器上的

如果用户访问的某个 rabbitMQ 没有真正的 queue  ，此机器会向真实保存 queue 的数据请求数据

节点类型：
1. 内存，所有的数据均存储在内存中
2. 磁盘，所有的数据均存储在硬盘中
>单节点必须为磁盘类型

负载策略：

>每台机器都可以设置

1. ha-mode：镜像队列的模式
- all 表示在集群中所有的节点上进行镜像（默认即此）；
- exactly 表示在指定个数的节点上进行镜像，节点的个数由 ha-params 指定；
- nodes 表示在指定的节点上进行镜像，节点名称通过 ha-params 指定。

ha-params：ha-mode 模式需要用到的参数
ha-sync-mode：进行队列中消息的同步方式，有效值为 automatic 和 manual。



![[rabbitmq-消息ack.png]]


![[rabbitmq-发送一条消息.png]]


![[rabbitmq-消费者与类库.png]]

![[rabbitmq-消费者流控.png]]