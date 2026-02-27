
# timer

```go
timer := time.NewTimer(2 * time.Seconds)
<-timer.C
```

>time.NewTicker  和 time.NewTimer 底层一样

```go
func NewTimer(d Duration) *Timer {
	c := make(chan Time, 1)
	t := &Timer{
		C: c,
		r: runtimeTimer{
			when: when(d),
			f:    sendTime,
			arg:  c,
		},
	}
	startTimer(&t.r)
	return t
}
```

- 创建了一个管道
- 创建一个 timer 结构体（把管道加进去）
- timer 内部，创建了一个 runtimeTimer 结构体
- startTimer 启动这个 timer ->  runtimeTimer

#### runtimeTimer：
- when ：定时器到期时间(当前时间 + 延迟时间)
- f : callback 函数，实际就是回调 上面新建的C，给管道发送信号

#### startTimer：
>源码中我没找到这个代码，是个接口，不知道具体哪里实现

- 调用 assignBucket，得到获取可以被插入的 timersBucket
- 调用 addtimerLocked 将 timer 插入到 timersBucket 中。从函数名可以看出，这同时也是个加锁操作。

#### timersBucket

runtime （1.13v）中，共有 64 个全局的 timersBucket
每个 timersBucket 负责管理若干个 timer（整个生命周期）：
- 创建
- 销毁
- 唤醒
- 睡眠

timersBucket 实际上内部是使用最小四叉堆（小根堆）来管理和存储各个 timer。
每个 timersBucket 都有一个对应的 timerproc 的 goroutine 来负责不断调度这些 timer

不断从堆底找寻到时间的 timer，移动 到期 timer 调用回调函数。实际是：给C管道写信号


# sleep

底层居然是跟 timer 一样的实现。

每次调用 sleep 的时候，都要创建一个 timer 对象。
需要一个 channel 来传递事件。

# tick sleep 区别

底层都一样，回调略有点区别：
- 给管道发信号，调试器来处理阻塞的G
- 挂起自己，等待到时间后，由P来调度G

tick 可以取消，sleep 不能主动取消


# 排它锁/互斥锁


>src/sync/Mutex.go

每次要操作一个数据时，要先获取该锁，否则等待。拿到锁的人，使用结束后，解锁

```go
var mutex sync.Mutex //创建一个锁变量
mutex.Lock()         //开始加锁
//......do something
mutex.Unlock()  //解锁
```


```go
type Mutex struct {
    state int32 //当前锁的状态
    sema  uint32 //信号量
}
//接口定义
type Locker interface {
  Lock()
  Unlock()
}
```


state 是一个 int32，被分成了4个区：

![[golang-锁-state-int32.png]]


|          |                                 |            |     |
| :------- | :------------------------------ | :--------- | --- |
| Locked   | 0 没有锁定，1已经被锁定                   |            |     |
| Woken    | 0 没有协程唤醒，1已经有协程唤醒，正在加锁过程中       | 是否有协程已经被唤醒 |     |
| Starving | 0 没有饥饿，1 饥饿状态，说明有协程阻塞了超过1ms     | 饥饿模式，不支持自旋 |     |
| Waiter   | 阻塞等待锁的协程个数，解锁时根据此值来判断是否需要释放信号量。 |            |     |

#### 自旋和阻塞

互斥场景下：只有一个 线程/进程 拿到锁并执行， 其他线程都是失败者. 
失败者的处理策略:

- 主动等待：进入不断重试（自旋模式）。可以减少线程切换造成的系统资源浪费, 这在多核处理器下才有意义
- 被动等待：阻塞当前进程(线程)切换到其他线程. 如果线程长时间的阻塞，其占用系统资源的造成的代价比线程切换更大时, 这个策略较有意义

>这里主要分析 就是：失败者的处理策略，实际就是：是否支持自旋


#### 混合模式

golang mutex 是：自旋 + 阻塞模式。(随时切换)

默认情况：自旋模式，乐观状态，每个协程都能很快执行完毕，只需要自旋一会即可。
如果发现，等待的协程过多，就会开启饥饿模式，通过队列管理等待的G。
如果发现，情况有所好转，再转回自旋模式

#### 自旋转条件：

- 锁被其它协程拿到，自己没拿到
- 普通模式(饥饿模式下，会直接交给等待队列的头部元素G)
- 当前运行在多核 CPU 环境中, 且 GOMAXPROCS >1
- 当前 Goroutine 进行自旋的次数小于 4
- 至少存在一个其他正在运行的处理器P, 并且它的本地运行队列(local runq)为空 (为了程序整体性能考虑)

>GO的自旋：runtime_doSpin 函数，执行了30次 PAUSE 指令以占用 CPU


#### 回到非饥饿模式：

如果一个 goroutine 获取到了锁之后，它会判断以下两种情况：
1. 它是队列中最后一个goroutine； 
2. 它拿到锁所花的时间小于1ms； 

以上只要有一个成立，它就会把锁转变回正常模式。

#### 切换到饥饿模式：

starvationThresholdNs ：值为1e6纳秒（1毫秒），当等待队列中队首 goroutine 等待时间超过 starvationThresholdNs 也就是1毫秒，mutex进入饥饿模式。

#### lock 加锁

1. 设置状态：state.Locked=1(CompareAndSwapInt32)，如果成功，直接返回
2. 设置失败，进入 slowLock 函数
2. 如果是正常模式(非饥饿模式) ：
	- 先尝试 自旋 4 次，如果均失败，进入下个模式
	- 如果这期间锁被释放了，重复第一步
2. runtime.Semacquire(&m.sema)，信号量-进入阻塞模式-等待唤醒
3. 循环此过程，直到可以加锁





死锁:函数1中设定了锁X，将X引用传到函数2中，2中直接锁，不释放，然后 return . 函数1执行流继续，这个时候，如果函数1想再加锁...阻塞



#### 解锁



默认=0：未使用
1=mutexLocked:锁定状态
2=mutexWoken:从正常模式被从唤醒
3=mutexStarving: 当前的互斥锁进入饥饿状态
4=waitersCount — 当前互斥锁上等待的 goroutine 个数


在 Lock() 之前使用 Unlock() 会导致 panic 异常

# 读写锁



如果一个协程加了读锁，另一个协程也可以加读锁，但不能加写锁了
如果一个协程加了写锁，其它协程不能加任何锁了

总结：一写多读

写锁优先集高于读锁，这种锁，适合读多写少的情况



- 读写比为 9:1 时，读写锁的性能约为互斥锁的 8 倍
- 读写比为 1:9 时，读写锁性能相当
- 读写比为 5:5 时，读写锁的性能约为互斥锁的 2 倍



