# 前置条件

1. 熟悉 TCP/IP
2. 了解 UDP
3. 有计算机网络基础

这里不做讲解了，有兴趣百度

> 但还是得致敬一下那些科学家吧，提供论文。

不讲UDP，下面所有的描述大部分是基于TCP，原因：

1. 99%的场景都是TCP，它更通用
2. UDP 仅适用于特定的场景：
    1. 帧同步，对传输速度有非常高的要求
    2. 音视频流，既想追求但速度传输文件过大，不过允许丢帧
3. UDP太复杂，因为要自实现 丢包重传、阻塞、面向连接等

# 概览

socket：网络中不同主机上的应用进程之间进行双向通信的实现

> 翻译：你如果想使用计算机网络传输东西，用这个 socket 就得了，它会给你露出几个API接口，直接调用，简单明了。

这里注意下，最早的商业化服务器 OS 是 unix，socket也是基于 unix 产生的，致敬下unix

> WIN/MAC 也是后来的产物，但UNIX更倾向专业人士，且没有 UI窗口化。

为啥一定要用socket?

> 它帮助你做了很多软硬件的工作。

1. 它有现成的代码/类库
2. 网卡如何收发数据
3. 数据传输中是否丢包
4. 双方以什么格式传输数据
5. 如何找到对方的地址
6. ......

> 个人觉得它最大的贡献是统一了规范\(前提是它确实好用\)，全世界的人都用它这一套代码规范，省了很多不必要的兼容/维护的麻烦

简单说说进程，它是由 OS 衍生而来，我们所有的程序，最终在 OS 层面，都是生成一个进程（分配内存、CPU执行、硬盘IO、权限等）。然后每个进程执行你自己的代码，可以进行网络通信。所以，从这个角度上来看，socket 也是进程间的通信。

> 进程间的通信不一定得走网络，还有IPC，有兴趣的自查可以

端口号，这东西，完全就是给网络用，不用网络的话，端口号没啥意义。给每个进程再标一个端口号，方便OS进行网络/进程 识别。

# socket 与 TCP/IP 的 关系

TCP/IP是协议规范，更像是论文，没到代码层面。而 socket 是通过代码实现了这套协议规范。

> 注：socket 不仅仅实现了 TCP/UDP， 还实现了IP层的一些协议。也可以理解为： socket 不完全等于 TCP/UDP

从实际角度上来看：当你使用 socket 后，基本就不太关心 TCP/UDP/IP 了，具体使用 TCP or UDP 只需要调用 socket 接口时，换个参数即可。

那么，实际上 socket 也可以理解为 操作 TCP/UDP/IP 协议的一个类库。

像：TCP的连接过程，IP路由等等这些，全被 socket 统一覆盖了

另外，我们听过 socket 编程，但很少听过 TCP 编程，我觉得从这个角度上来看，也是 socket 实现了 tcp/ip 协议。并在此基础上，可以做些编程如：自定义协议开发

从分层的角度看，socket 更倾向于在传输层的一系列操作。应用层是：HTTP FTP 这些软件，物理层是网卡/网线在操作。

# 大概的通信流程

1. C/S端请用 socket 公共函数
2. 内核的一些主要功能及函数

![[socket概览.png]]

> 从这里我们也看出，用户态的做的事情非常少，主要还是 socket 帮助我们做了大部分的事情。
> 再换个角度看，其实更像是：两台机器 OS 内核间的通信

# socket 编程 函数概览

> 这里以C语言函数为标准

先看下大概流程图：

S端

| 函数名    | 描述                 | 备注  |
| ------ | ------------------ | --- |
| bind   | 此进程想要绑定当前机器的哪个端口号  |     |
| listen | 开始监听这个 端口号发过来的网络操作 |     |
| accept | 接收连接成功后的FD         |     |

C端

|函数名 |描述         |备注       |
|-------|-------------|-----------|
|connect|与S端建议连接|完成3次握手|

C/S共用

|函数名 |描述                                    |备注        |
|-------|----------------------------------------|------------|
|socket |创建一个 socket 结构体，并初始化基础字段|            |
|close  |                                        |            |
|read   |读取数据                                |阻塞        |
|write  |发送数据                                |阻塞        |
|recv   |读取数据                                |可选择不阻塞|
|send   |发送数据                                |可选择不阻塞|
|readv  |读取数据，可读入多个缓冲区              |            |
|writev |发送数据，可写入多个缓冲区              |            |
|recvmsg|读取数据                                |            |
|sendmsg|发送数据                                |            |


所有函数大体被分成了两类：

1. 面向连接（收发数据之前的连接、TCP/IP协议的基础）
2. 面向IO（C/S端主要收发数据的函数）

最核心的函数是：socket ，不管 C 还是 S ，且必须在最开始的时候，调用这个函数，后续所有的操作都要基于这个 函数返回值

我们看到：最终，还是 IO 函数 最多，基本上也就确定了，程序员日常操作都集中在IO层面。

# socket 函数

主要是创建一个 socket FD\(根据用户态的参数信息\)，大致如下：

1. 创建一个 scoket 结构体 并填充数据
2. 创建一个 sock 结构体 并填充数据，与socket结构体做关联
    1. 填充该结构的基础信息，如：IP、端口、协议类型等
    2. 给该结构体设置状态，如：连接状态
    3. 找到协议栈的代码，继续填充，如：TCP 的 SYN 包发送的头信息等
3. 创建 inet\_sock：TTL,组播列表，IP地址，端口

> 两端的基础网络信息放在这里\(尽量与协议无关\)

1. 创建 inet\_connection\_sock：面向连接的一些属性
2. 创建 tcp\_sock，TCP的一些专有属性，如：滑窗、拥塞控制

> tcp\_sock 继承 inet\_connection\_sock 继承 inet\_sock 继承 sock

用户态最终拿到的是 socket FD ，而实际大部分真实的操作是基于 sock 结构体

这里主要分析下：socket 和 与 sock 结构体

```
type  socket struct {
	short         		type;  //套接字所用的流类型
	socket_state  		state; //套接字所处状态
	long          		flags; //标识字段，目前尚无明确作用
	struct proto_ops  	*ops;  //操作函数集指针
	struct wait_queue 	**wait;//等待队列
	struct inode      	*inode;//inode结构指针
    struct sock		    *sk;
    struct socket_wq __rcu *sk_wq; ///套接字的等待队列
};
```

> socket 结构看着挺简单，它主要是透给用户态的，而 sock 结构是给内核态的

proto\_ops 就是你选择了什么协议类型，就找到该协议的一组操作函数

sk\_wq： epoll就是挂载在这个上面，一但读写缓冲区就绪，就会回调

inode 主要是把 socket 结构体与硬盘绑定最终以FD形式返回

```
enum sock_type
{
   SOCK_STREAM  = 1, // 用于与TCP层中的tcp协议数据的struct socket 
   SOCK_DGRAM   = 2, //用于与TCP层中的udp协议数据的struct socket 
   SOCK_RAW     = 3, // raw struct socket
   SOCK_RDM     = 4, //可靠传输消息的struct socket
   SOCK_SEQPACKET = 5,// sequential packet socket
   SOCK_DCCP    = 6,
   SOCK_PACKET  = 10, //从dev level中获取数据包的socket
};
```

> 感觉更像是协议类型，如：TCP/UDP

```
typedef enum {  
    SS_FREE = 0,            //该socket还未分配  
    SS_UNCONNECTED,         //未连向任何socket  
    SS_CONNECTING,          //正在连接过程中  
    SS_CONNECTED,           //已连向一个socket  
    SS_DISCONNECTING        //正在断开连接的过程中  
}socket_state;
```

> socket状态。这里发现，并没有 ESTABLISHED CLOSEING FIN\-WAIT\-1 常见状态，因为这些状态是专属于 TCP 协议的。而实际上这个状态才更通用化的。

```
struct socket 中的 flags 字段取值：
  #define SOCK_ASYNC_NOSPACE  0
  #define SOCK_ASYNC_WAITDATA 1
  #define SOCK_NOSPACE        2
  #define SOCK_PASSCRED       3
  #define SOCK_PASSSEC        4
```

sock 结构体 因为太大，列出主要的吧：

```
struct sock_common	__sk_common;	 // 网络层套接字通用结构体

socket_lock_t sk_lock;// 套接字同步锁
  
struct sk_buff_head	sk_receive_queue;// 收到的数据包队列
struct sk_buff_head	sk_write_queue;  // 发送队列/重传队列
struct sk_buff_head back_log;//当套接字忙时，数据包暂存这里

union {
    struct socket_wq __rcu    *sk_wq;//进程等待队列
    struct socket_wq    *sk_wq_raw;
};

unsigned long   sk_lingertime;//连接关闭前未发送数据的总数
struct sk_buff_head sk_error_queue;//错误链表，存放详细的出错信息。
long            sk_rcvtimeo;//读超时
long            sk_sndtimeo;//写超时
struct timer_list    sk_timer;//定时器，像3次握手等使用

    
```

sock 结构体 才是整个 网络传输的 核心，这里主要功能：

1. TCP 稳定控制：重传、滑窗、阻塞、排序
2. 收发数据
3. TCP 连接控制：建立连接、关闭连接、连接状态控制

> 一般来说 内核控制 sock 结构体，用户态控制的是 socket 结构体

sk\_lock:用户态读数据得加锁，因为这期间可能内核收到网卡信息，要写入缓存队列。另外，状态发生变更时，内核也会加锁。

sk\_wq:进程等待连接、等待输出缓冲区、等待读数据时，都会将进程暂存到此队列中

sk\_lingertime:这个参数引发一个问题，连接如果关闭，而还有数据未发送完，如何处理？好像内核有个参数，你开启它就等待一段时间 你关闭他就直接把数据丢了，非优雅关闭。

sk\_rcvtimeo:这个参数有意思。socket 本身就是阻塞模式\(epool seelect recv阶外\)的，像：accept/connect/send/recv这些都是阻塞/半阻塞的。如果设置这个参数就可以非阻塞。但感觉可能还是有性能的牺牲或者得会用，知道怎么用，不然还是默认值吧。

sock\_common

```
union {
        __addrpair    skc_addrpair;
        struct {
            __be32    skc_daddr;//远端地址 IP4本地地址
            __be32    skc_rcv_saddr;//本地地址 IP4本地地址
        };
};

unsigned short      skc_family;         //地址族
volatile unsigned char  skc_state;      //连接状态
unsigned char       skc_reuse;          //SO_REUSEADDR设置
int         		skc_bound_dev_if;
struct hlist_node   skc_node;
struct hlist_node   skc_bind_node;      //哈希表相关
atomic_t        	skc_refcnt;         //引用计数
```

这里是些基础的信息，还有连接状态等。sock 结构体也算是继承了些结构体吧

#### socket sock 结构体 小结

其实，unix/linux 内核在实现 socket 中还有N多个结构体，还牵扯到 网卡、驱动、中断机制等等吧，我这里也只是列出2个关键的结构，太细节的我也不懂，也没时间看。

但是从 sock 这个结构体来分析看，它其实就是把网络进行又一次分层，封装，达到最终只露出几个公共接口给程序员使用。

其中的核心就是：通过几个发/发缓冲数据队列，内核再睡眠/唤醒进程，就实现了....

# 数据收发队列

它主要是包含了4个队列：

1. backlog：用户正在占用 socket ，可能正在读操作，数据暂时放到 backlog，如果满了就直接丢弃数据。数据：可能乱序，还有TCP头信息
2. prequeue:
3. sk\_receive\_queue：最干净的数据，没有TCP协议头、顺序正确、校验过的，用户可直接使用
4. out\_of\_order：乱序报文

> 以上队列是有优先集之分的，级别越高，越先处理。也就是序号大的

正常情况

> 内核肯定是希望来的数据都是正常的，直接扔到 sk\_receive\_queue 中，用户直接可以取走的。

数据乱序情况

1. A给B~发送3段报文 1 2 3
2. 报文3先到的，用户态希望的是报文1，所以 不能直接读取。把3报文扔到 out\_of\_order 中
3. 报文1 来了，2也来了，都扔到 sk\_receive\_queue 中
4. 从 out\_of\_order 中把3取出来，扔到 receive\_queue 中。

socket 被用户态给锁住了

> 用户态正在操作 socket 读数据中，此时有消息过来了，直接扔到 backlog 中

读数据阻塞中

1. 用户态开始读数据，此时没有数据了，它就放弃锁，然后，进入睡眠状态，等待新数据过来 。
2. 此时，有新数据来了，因为用户没加锁，也没有正在读，新来的数据不会进入backlog。这条数据会插入到 prequeue 中，
3. prequeue 有新数据会唤醒该进程，唤醒后立刻加锁
4. 先读 sk\_receive\_queue 数据 ，再读 prequeue ，最后读 backlog

总结：

通过一个锁，用户态读取操\(可能会操作 sk\_receive\_queue prequeue \)作就会加锁，新来的数据就得进入 backlog。用户态没有加锁的时候，内核就可以直接操作 sk\_receive\_queue \+ out\_of\_order。而用户态没加锁但是阻塞模式下，就是操作 prequeue。

# bind 函数

把要监听 的IP 端口号，更新到 sock 结构体内，listen 函数执行之前必须得调用 bind 函数

# listen 函数

1. 找到 sock 结构 ，
2. 找到 inet\_connection\_sock 结构 ，找到 icsk\_accept\_queue 成员
3. 创建 request\_sock\_quue
    1. 创建半连接队列
    2. 创建全连接队列

request\_sock\_quue 队列

```
struct request_sock_queue {
    struct request_sock	*rskq_accept_head;
    struct request_sock	*rskq_accept_tail;
    rwlock_t		syn_wait_lock;
    u8			rskq_defer_accept;
    /* 3 bytes hole, try to pack */
    struct listen_sock	*listen_opt;
};
```

request\_sock 全连接队列

listen\_sock 半连接队列，关键参数参数：backlog ，半连接队列最大长度

两个队列里存储的成员都是：requset\_sock 结构体

```
struct request_sock {
	struct sock_common		__req_common;
    ......
};
```

没用的我就没记，其中好像就一个是值得说的：sock\_common ，基本上它就是一个小型时 sock 结构体，也能理解，毕竟还到正式使用，甚至可能还是半连接状态 ，没必要创建一个完整的 sock 结构体。它只要能表示清楚：要连接方的基础信息5元组即可。

当半连接完成后，进入 全连接后，既：把这个成员从半连接队列转移到全连接状态后。 accept 函数就能拿到此socket fd ，但在这这前，内核会把这个 request\_sock 进行完整化，如：创建 socket sock 结构体，绑定FD等等。

# accept

从处于 established 状态的连接队列头部取出一个已经完成的连接，如果这个队列没有已经完成的连接，accept\(\)函数就会阻塞，直到取出队列中已完成的用户连接为止。另：此函数会返回一个新的FD

这里很有趣，我们S端正常要监听，得主动创建一个 socket FD，但是 accept 会源原不断的返回新的 socket FD ，有啥区别？

# 读写缓存区

内核给每一个TCP套接字都有一个发送缓冲区

SO\_SNDBUF控制大小。调用send发送数据时\(默认阻塞模式\)，如果缓冲区没满，调用直接返回。系统内核在IP层发送数据的时候，并不是按照我们调用send接口发送的数据包大小来进行发送，即使我们调用send发送的数据大小小于1460字节\(MTU\-TCP首部\-IP首部\)。因为我们调用send接口实际是将数据复制到缓冲区中，而内核基本上是按照最大MSS大小\(1460字节\)从缓冲区中取数据发送出去，当缓冲区中数据小于MSS，则将剩余数据全部发送出去。TCP的发送缓冲区必须为已发送的数据保留一个副本，直到它被对端确认为止，才能从缓冲区中删掉已确认的数据。

内核给每一个TCP套接字都有一个接收缓冲区，

可以通过SO\_RCVBUF套接字选项来更改。接收缓冲区被TCP用来保存接收到的数据，直到应用程序来读取。对于TCP来说，接收缓冲区中可用空间的大小限定了TCP通告对端的窗口大小。TCP套接字的接收缓冲区不能溢出，所以发送端不能发送超过接收端通知的窗口大小，否则在接收端将丢弃数据包。收端通知发端，接收窗口关闭（win=0），从而保证了TCP是可靠传输

以上两个都以队列维护

# 阻塞

先区分4个概念：

1. 阻塞\(blocking\)：程序执行到某个点~等待着结果返回不动了，进程被挂起。如：accept、recv等
2. 非阻塞\(non\-blocking\)：执行到某个点，立刻返回，不管其它的。
3. 同步\(synchronous\)：程序执行到某个点~等待着结果返回不动了，进程依然处于活动状态
4. 异步\(asynchronous\)：执行到某个点，立刻返回，不管其它的。

往底层次看的话，阻塞其实是：当进程的代码，被调度到 CPU上，开始执行。在执行的过程中，发生了睡眠，如：sleep http\-request 等，比较耗时的操作或主动睡眠，此时 OS 就把该进程扔到睡眠队列中，更新进程状态。让出 CPU 执行权限，给到其它进程继续执行。

所以阻塞这东西，更像是OS为了平衡进程执行时间而产生的。

> 当然也有主动的，如：SLEEP 这种不讨论

# epoll

sk\_wq

## 总结

为什么一定要内核介入？
因为 网络IO过于复杂，牵扯了大部分计算机资源，包括硬件，如网卡。需要的权限过多，内核直接来参与更方便。最终，通过通过 socket API 给到用户
从用户的角度看，socket 更像是一个文件的IO，这也是LINUX哲学的：万物皆文件。收消息 是读 ，发消息是写。

另外，从多协议/分层 来看，不管什么协议最终要实现的目标就是：收/发消息。
