数据存储 + 分布式  系统

若干个 zk 节点组成一个 zk 集群，集群中有半数未挂，就依然还能提供服务

具体的服务：
1. 提供数据存储
2. 可监控某一个数据
3. 数据有失效时间

# 数据存储

数据的存储是以树的形势进行组织的~

树的每个节点叫：znode

znode的类型：
- 永久目录结点
- 永久目录结点-顺序带编号
- 临时节点
- 临时节点-顺序带编号

# 应用场景

动态配置信息：服务的注册与发现
>服务的启动与宕机是不固定的，当启动时，便去 zk 里创建一个临时节点，一但宕机，连接断了，该节点也消失了。对于客户端，监听某个目录下的这些动态配置信息，一但发生改变，也会第一时间做选择

分布式锁：A创建一个临时节点，证明给某个业务加锁了，B也想操作该业务，但发现该节点值存在，不能加锁，进入等待，并监听该节点，一但A结束操作，删除了该节点。B就可以加锁并操作了


# 选举

Zookeeper 会在以下场景进行选举：
1、Zookeeper 集群启动初始化时进行选举
2、Zookeeper 集群 Leader 失联时重新选举
>Leader 故障后，余下的非 Observer 服务器都会将自己的服务器状态变更为LOOKING


server_id：服务器ID。用来唯一标识一台ZooKeeper集群中的机器，每台机器不能重复
zxid：事务ID。用来标识一次服务器状态的变更（任何数据的变更+1）
- Epoch 任期：完成本次选举后，直到下次选举前，由同一 Leader 负责协调写入
- 事务计数器：单调递增，每生效一次写入，计数器加一
>Epoch：逻辑时钟。也叫投票的次数，同一轮投票过程中的逻辑时钟值是相同的，每投完一次票这个数据就会增加。

![[zk_zxid.png]]


每个节点的状态：

- LOOKING: 竞选状态
- FOLLOWING: 随从状态，同步 leader 状态，参与投票
- OBSERVING: 观察状态，同步 leader 状态，不参与投票
- LEADING: 领导者状态

![[zk_选举.png]]

1. server_id = 1 ,zbix = 0 ,发起候选，并选自己为 leader ， (0,1) 状态为：looking ，发给另外2个 zk 节点
2. server_id = 2 ,zbix = 0 ,发起候选，并选自己为 leader ， (0,2) 状态为：looking ，发给另外2个 zk 节点
3. server_id = 3 ,zbix = 0 ,发起候选，并选自己为 leader ， (0,3) 状态为：looking ，发给另外2个 zk 节点

- server_id = 1，(0,2) (0,3) ，对比 Epoch zxid server_id ，下轮投票： (0,3)
- server_id = 2，(0,1) (0,3)，对比 Epoch zxid server_id ，下轮投票： (0,3)
- server_id = 3，(0,1) (0,2)，对比 Epoch zxid server_id ，下轮投票： (0,3)

1. server_id = 1 ,zbix = 0 ,发起候选，并选自己为 leader ， (0,3) 状态为：looking ，发给另外2个 zk 节点
2. server_id = 2 ,zbix = 0 ,发起候选，并选自己为 leader ， (0,3) 状态为：looking ，发给另外2个 zk 节点
3. server_id = 3 ,zbix = 0 ,发起候选，并选自己为 leader ， (0,3) 状态为：looking ，发给另外2个 zk 节点

- server_id = 1，(0,3) (0,3) ，对比 Epoch zxid server_id ，下轮投票： (0,3)
- server_id = 2，(0,3) (0,3)，对比 Epoch zxid server_id ，下轮投票： (0,3)
- server_id = 3，(0,3) (0,3)，对比 Epoch zxid server_id ，自己就是3：变更状态为：LEADING，并告知另外两个我是新的LEADING

每轮投票，每个节点都会收到相关的选票，然后根据选票内容，开始比对：
Epoch zxid Epoch，计算出哪个票的值最大，下轮的选票就是这张。
如果发现，选票已经过半了，且自己就是那个主角，就变更状态为：LEADING，并通知其它节点
其它节点收到有新 LEADING ，变更自己的状态。


# 节点角色

leader：只负责消息同步和写操作，
follower：负责读，和把写转到 leader
observer也可以接收client的读操作，写操作依然转发到 leader。
>读的请求会大于写


# 同步/zab

集群间通过：ZAB广播通信。

Zookeeper Atomic Broadcast 协议：当leader挂了，它便进入恢复模式，开始选择新的 leader，选过多后，进入广播模式。开始同步数据。也就是说，所有数据在每台服务器中都有一份副本。

当 client 发起写请求后：

1. leader 接收写请求
>如果当前请求节点是 folower ，会将请求转发至 leader
>2. 写入成功后 leader 会等待所有的 folower 返回 ack 消息
>3. 当 ack 消息过半的时候，leader 就发消息告诉 所有  folower 进行 commit





如果集群，那就牵涉到事务唯一，它是通过 zxid，全局唯一号，也就是事务号。

ISR：a set of in\-sync replicas


# 安装
docker pull wurstmeister/zookeeper

//启动zk

docker run \-rm \-d \-\-name zookeeper \-p 2181:2181 \-v /etc/localtime:/etc/localtime wurstmeister/zookeeper

