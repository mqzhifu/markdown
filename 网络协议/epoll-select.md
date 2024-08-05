

# IO 多路复用

假设 需要 IO 的 FD 很多，期间某 FD 还有阻塞的可能，如果一个一个处理，性能太低。最好能有一个代码库，（单 进程）可同时监听这些FD，哪个能读了，就处理哪个，哪个能写了就处理哪个，避免阻塞。

> 我目前能想到的使用声场景就只有：网络 socket 编程 ，因为：客户端的连接是不确定，是肯定牵扯阻塞的。

如果使用多进程可解决阻塞，但是代码难度增加，进程开销增加。

# socket 编程大的方向

1. 最基本的 socket 编程（阻塞式）
2. 多进程/多线程（非阻塞式）
3. IO 多路复用（非阻塞/单进程）
    1. select/poll
    2. epoll/kqueue

# select/poll

select 是 C 语言 的一个标准库，用于实现 IO多路复用。（出现时间最早）
其核心就是：把若干你想要操作的IO\-FD，扔到某个集合中，然后轮询监听这些FD，不阻塞。只需要一个进程就能解决所有问题。



函数原型：

```
int select(int maxfdp,fd_set *readfds,fd_set *writefds,fd_set *errorfds,struct timeval *timeout);
```

函数分析，告诉内核：我有3个文件 FD 集合

1. 最大的 FD
2. 读操作
3. 写操作
4. 异常操作
5. NULL：阻塞， 0：不管是否有可IO的FD，直接返回，不阻塞。 具体时间：到达一定时间没有事件发生，结果阻塞

继续告知内核：等待多久后，返回就绪 FD 的个数

集合是个 bitmap 结构

清某个集合清空
>FD_ZERO( fd_set * fdset  )

删除集合中某个FD，某一位置 0
>FD_CLR(fd_set  * fdset)

向某个集合添加 FD:某一位置  1
> FD\_SET\(serverSocket, &read\_set\);

判断某个集合中的某个FD，是否可操作： 某一位置 是否存在 
> FD\_ISSET\(serverSocket, &read\_set\)

```c++
socket s;
int ret = 0;
.....
struct timeval tv;
fd_set set;
while(1)
{
    FD_ZERO(&set);//将你的套节字集合清空
    FD_SET(s, &set);//加入你感兴趣的套节字到集合,这里是一个读数据的套节字s
    tv.tv_sec = 0;
    tv.tv_usec = 100000;//100微秒

    ret = select(s+1,&set,NULL,NULL,&tv);//检查（s+1个）套节字是否可读
	if(ret > 0)
	{
	    if(FD_ISSET(s, &set) //检查s是否在这个集合里面,
	    { 
		    //select将更新这个集合,把其中不可读的套节字去掉
		    //只保留符合条件的套节字在这个集合里面
		    
		    //do something here
	    }		
	}	    
}

```

每次都要执行函数：FD_ZERO
每次都要遍历整个集合，且遍历3次：读( 正常读+accept )、写、异常


![[socket_select.jpg]]

当网卡收到数据后，给 CPU 发中断信号。然后，找到某个 socket 后，会唤醒此进程，但仅仅是唤醒，并不告诉具体是：读、写
而进程被唤醒后，不知道是读还是写，还需要重复遍历整个集合。

另外，select 函数在执行时，要把集合 从用户态复制到内核态。 当 select 结束后，有数据来，还要把集合从内核态复制到用户态

优点：

1. 所有平台都支持
2. 实现了IO多路复用

缺点：

1. 单进程可打开的 FD 有限制。间接说明 select 可管理的 socket FD 是有上限的
2. 轮询这个实在有点蛋疼，一次循环遍历的成本太高。
    1. 主动 探测 FD是否可 IO。
    2. 只返回就绪 FD 的总数，并不具体到某个FD上。
    3. 每次都要重置 FD\_SET，也就是用户态数据CP到内核态。
    4. 一但高并发，FD\_SET 很大。但活跃的FD只占少数，但是循环遍历整个FD\-SET，很可怕。
3. 它不是内核态管理 FD集，而是需要用户态复制到内核态

> 如果能主动回调，就减少了主动轮询，效率应该更好些。

poll跟它差不多，算是对 select 的升级版。只是 FD 的保存集合 略有差别，它能支持 更多的 FD\_SET 数量。

这两个的时间复杂都是：O\(n\)

# epoll

> unix 下叫 kqueue


![[socket_epoll.jpg]]

eventpoll

#### 创建：epoll。

> epoll\_create\(int size\)

有3个关键成员变量：
1. rbr 红黑树 (存储所有需要监听的FD)
2. rdllist 链表（存储已就绪的FD ）
3. wait_queue_head_t  ，如果是阻塞模式，把当前进程及回调函数注册进去

#### 注册事件

> epoll\_ctl\( int epfd, int op, int fd, struct epoll\_event \*event\)

#### 等待事件返回

> epoll\_wait\(int epfd, struct epoll\_event \*events, int maxevents, int timeout\)

```

struct epoll_event  ev, events[MAX_EVENTS];

epollfd = epoll_create1( 1 );

ev.events   = EPOLLIN;
ev.data.fd  = listen_sock;

epoll_ctl( epollfd, EPOLL_CTL_ADD, listen_sock, &ev )

for (;; ){
    nfds = epoll_wait( epollfd, events, MAX_EVENTS, -1 );
    
}

```

> ET：句柄在发生读写事件时只会通知用户一次
> LT：只要句柄一直处于可用状态，就会一直通知用户。

优点：

1. 事件驱动\(回调\)，不用轮询了
2. 内核态直接管理 FD 集合
3. 对 FD集合的搜索，增加了 红黑树 算法

缺点：

1. 仅限 LINUX使用

> ni

时间复杂度：O\(1\)

## 其它
