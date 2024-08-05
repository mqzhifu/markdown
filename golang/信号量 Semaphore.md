# 概览

进程/线程-并发操作一个资源时，做排队处理，同一时间只能有一个进程可访问该资源。
释放后，从队列中取出一个进程/线程继续操作该资源

官方的解释：给每一个进程一个信号量，代表每个进程当前的状态，未得到控制权的进程，会在特定的地方被迫停下来，等待可以继续进行的信号到来。


# 注意

golang 中 Semaphore 是底层使用，并没有直接的API给程序员使用。

但，golang 官方给了一个扩展包：golang.org/x/sync v0.7.0，可以下载，使用与学习

# Semaphore

#### 创建一个信号量容器：

```go
// 创建资源数为 n 的信号量
func NewWeighted(n int64) *Weighted {
 w := &Weighted{size: n}
 return w
}
```

#### 查看 Weighted  waiter 结构体：

```go
type Weighted struct {
// 最大资源个数,取走时会减少，释放时会增加
 size    int64
 cur     int64      // 计数器，记录当前已使用资源数
 mu      sync.Mutex   // 互斥锁，对字段保护
 //等待队列，表示申请资源时由于可使用资源不够而陷入阻塞等待的调用者列表
 waiters list.List  
}
```

```go
type waiter struct {
 n     int64        // 调用者申请的资源数
 ready chan<- struct{}  // 当调用者可以获取到信号量资源时, close chan，调用者便会收到通知，成功返回
}
```

#### 获取一个资源：
```go
func (s *Weighted) Acquire(ctx context.Context, n int64) error {
	s.mu.Lock()
	// 有可用资源，直接返回
	if s.size-s.cur >= n && s.waiters.Len() == 0 {
		s.cur += n
		s.mu.Unlock()
		return nil
	}
	
	// 程序执行到这里说明无足够资源使用
	
	if n > s.size {
		s.mu.Unlock()
		<-ctx.Done()
		return ctx.Err()
	}
	
	
	// ready channel 用于通知阻塞的调用者有资源可用，由释放资源的 goroutine 负责 close，起到消息通知的作用
	ready := make(chan struct{})
	// 资源不足，构造 waiter，将其加入到等待队列
	w := waiter{n: n, ready: ready}
	elem := s.waiters.PushBack(w)      // 加入到等待队列
	s.mu.Unlock()
	
	// 调用者陷入 select 阻塞，除非收到外部 ctx 的取消信号或者被通知有资源可用
	select {
		case <-ctx.Done():     // 收到外面的控制信号
			err := ctx.Err()
			s.mu.Lock()
			select {
			// 再次确认是否可能是被唤醒的，如果被唤醒了则忽略控制信号，返回 nil 表示成功
				case <-ready:    
					err = nil
				// 收到控制信息且还没有获取到资源，就直接将原来添加的 waiter 删除掉
				default:      
					isFront := s.waiters.Front() == elem     // 当前 waiter 是否是链表头元素
					s.waiters.Remove(elem)     // 删除 waiter
	// 如果是链表头元素且有资源可用则尝试唤醒链表第一个等待的 waiter				
					if isFront && s.size > s.cur {    
						s.notifyWaiters()
					}
			}
			s.mu.Unlock()
			return err
		case <-ready:      // 消息通知，请求资源的 goroutine 被释放资源的 goroutine 唤醒了
		return nil
	}
}
```

#### 异步获取一个资源：

```go
func (s *Weighted) TryAcquire(n int64) bool {
	s.mu.Lock()
	success := s.size-s.cur >= n && s.waiters.Len() == 0
	if success {
		s.cur += n
	}
	s.mu.Unlock()
	return success
}
```

#### 释放资源
```go
func (s *Weighted) Release(n int64) {
	s.mu.Lock()
	s.cur -= n       // 释放占用资源数
	if s.cur < 0 {
		s.mu.Unlock()
		panic("semaphore: released more than held")
	}
	
	s.notifyWaiters()   // 唤醒等待请求资源的 goroutine
	s.mu.Unlock()
}
```

```go
func (s *Weighted) notifyWaiters() {
	for {
		next := s.waiters.Front()     // 获取队头元素
		if next == nil {        // 队列里没有元素
			break
		}
	
		w := next.Value.(waiter)
		if s.size-s.cur < w.n {       // 资源不满足请求者的要求
			break
		}
		
		s.cur += w.n              // 增加已用资源
		s.waiters.Remove(next)
		close(w.ready)         // 关闭 ready channel，用于通知调用者 goroutine 已经获取到资源，继续运行
	}
}
```


#### 代码逻辑分析

其实不难：就是看一下 资源 N 是否够用，如果够用，就直接返回，如果不够用，就创建一个 ready 信号结构体，加到等待队列中，然后阻塞监听此信号。如果其它持有者释放了该资源，就从队列中挑一个 waiter ，并发送信号 ，该  witer 就可以拿到资源了。

#### 信号量与普通的排它锁-区别：
1. 更强调的是进程/线程级别的操作
2. 有排队的机制
3. 进程/线程会阻塞
4. 有信号通信，接到唤醒操作，阻塞结束



#### 小结

GO自己实现了一套类似  sleep/wakeup 机制（进程/线程并发读写资源）。也算是帮程序员省了很多代码。


#  golang 内部机制


Semaphore 包，是不开放给程序员使用的。可以使用的 Semaphore 包是 sync 下面的，
GOLANG 其内部的实现还有一套详细的规则

runtime 内部定义了一个大小为 251 的全局变量 semtable 数组，来管理所有的 Semaphore。
```go
const semTabSize = 251

var semtable [semTabSize] struct {
	root semaRoot // 平衡树的根
	pad  [cpu.CacheLinePadSize - unsafe.Sizeof(semaRoot{})]byte // 填充位
}
```

> semTabSize : goroutine ID

semaRoot 就是一个节点，也算是树的开头结点，由此节点，可以找到下一个节点，即：
每 semtable 都包含一颗 树

```go
type semaRoot struct {
    lock  mutex
    treap *sudog // 平衡树的根节点
    nwait uint32 // 挂起等待的 goroutin 的数量
}
```


```go
type sudog struct {
	// 以下字段受hchan保护
	g *g

	// isSelect 表示 g 正在参与一个 select, so
	// 因此 g.selectDone 必须以 CAS 的方式来获取wake-up race.
	isSelect bool
	next     *sudog
	prev     *sudog
	elem     unsafe.Pointer // 数据元素（可能指向栈）

	// 以下字段不会并发访问。
	// 对于通道，waitlink只被g访问。
	// 对于信号量，所有字段(包括上面的字段)
	// 只有当持有一个semroot锁时才被访问。
	acquiretime int64
	releasetime int64
	ticket      uint32
	parent      *sudog //semaRoot 二叉树
	waitlink    *sudog // g.waiting 列表或 semaRoot
	waittail    *sudog // semaRoot
	c           *hchan // channel
}

```


分析大体逻辑：
- 当有多个 G  并发访问某一个资源时，同一时间只能有一个 G 获取到，未获取到的 G 进入阻塞状态。
- gopark()
- 阻塞状态的 G ，会修改当前状态，将创建一个 sudog 结构，与 G关联
- sudog 会被添加到 semtable 中
- 等某个 G 释放了资源，会人 semtable  找一个等待的 G
- goread() ，等待 P 重新调度 执行

分析：就是当出现多个 G 访问一个资源时，对 G 进行睡眠与阻塞。


sudog 像是：对 G 的又一层封装
