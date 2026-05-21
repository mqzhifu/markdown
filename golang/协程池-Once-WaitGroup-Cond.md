# 协程池

实现：
- 外层：控制当前开了多少协程、创建协程、销毁协程
- 内层：根据用户传的参数、context、函数地址，去执行这个协程


控制上限：内存泄露
复用资源：减少GC
观测性与监控告警
公共的defer recover：某一个函数pinic ，导致整个进程全挂



# 协程栈

初始栈大小  2KB，最大上限：1GB

stackMin   = 2048  // 初始2KB
stackGuard = 8192  // 警戒区8KB
stackMax   = 1GB   // 单栈最大

每个 goroutine 一个整块、连续的栈

触发扩容（预判式）：编译器在函数入口插检查

线程栈通常 1–8MB

# Once 原理

一个函数，即使多协程同时调用 ，也只执行一次。
主要：用于一些包的初始化工作。还有，单例模式。

type Once struct {
    done uint32
    m    Mutex
}

done:标记，已经函数已经执行过一次了
m: 协程安全

它的内部还有一个双检测机制
```
func (o *Once) Do(f func()) {

	if atomic.LoadUint32(&o.done) == 1 {
		return
	}

	o.m.Lock()
	defer o.m.Unlock()

	if o.done == 0 {
		f()
		o.done = 1
	}
}
```

问题：因为内部有悲观锁，会阻塞后面的协程

Once 为什么不用 CAS 完成？
CAS 无法保证初始化完成的可见性

# Wait group

```
type WaitGroup struct {
	noCopy noCopy      // 防拷贝检查（不允许把 WaitGroup 当参数值传递）
	state  atomic.Uint64 // 💡 高级黑科技：一个 64 位的原子无符号整型
	sema   uint32        // 💡 信号量：底层的等待队列钥匙
}
```



state:
高32位：count，还剩多少任务。add(1)就count+，done(),这里就减count-1
低32位：有多少 goroutine 正在 Wait()。

加减过程（add done wait）中：
- 当count = 0 and wait > 0，会唤醒 wait 协程，并结束
- 当count=0 and wait = 0 结束
- 当 count > 0 时，该协程进入等待状态

为什么一个变量拆成2个部分？
一个原子变量同时管理两种状态，一次可以修改2种值，减少锁


经典错误：
- Add 写在 goroutine 里面，Wait 可能提前结束。它两有竞争关系
>子协程还没来得急add(1)，主协程wait了，发现 count=0 & wait = 0 ，直接结束
- Add(-1) 使用负数
- 不要直接 copy WaitGroup
- 不要复用 WaitGroup

为什么不用 channel 实现？
- 少对象
- 少调度
- 少 channel 操作

应用场景：开多个子任务，同时完成一个大任务
不适合场景：控制协程数

正常得有for select default+sleep，代码量大，容易出错，且需要定义的变量多且还得循环。 waitGroup靠sema+semaphore 原理，是真的挂起当前协程，最后再被唤醒



# COND


创建 Cond：使用 sync.NewCond(&mu) 关联互斥锁。
等待 (Wait)：在 L.Lock() 和 L.Unlock() 之间，使用 c.Wait() 暂停协程。
通知 (Signal/Broadcast)：状态改变后，使用 c.Signal()（唤醒一个）或 c.Broadcast()（唤醒所有）。 

核心：可以多协程同时等待一个变量的值，如果等待不到，就挂起当前协程。
外部可以：改变这个变量的值，然后，可以通知一个/全部等待的协程。


```
func main() {
	var mu sync.Mutex
	cond := sync.NewCond(&mu)
	resName := "" // 共享资源

	// 启动 3 个工作协程，都在等同一个资源
	for i := 1; i <= 3; i++ {
		go func(id int) {
			mu.Lock()
			// 必须用 for，防止虚假唤醒
			for resName == "" { 
				fmt.Printf("协程 %d：资源没下载完，我先睡了...\n", id)
				cond.Wait() // 释放锁，挂起；被唤醒后，重新抢锁
			}
			// 抢到锁且条件满足，执行业务
			fmt.Printf("协程 %d：检测到资源 [%s]，开始处理！\n", id, resName)
			mu.Unlock()
		}(i)
	}

	// 主协程模拟下载资源
	time.Sleep(2 * time.Second)
	mu.Lock()
	resName = "Golang_v1.26_Patch" // 改变条件
	fmt.Println("\n【主协程】：资源下载完成，广播通知所有协程！\n")
	cond.Broadcast() // 一枪令下，全部唤醒
	mu.Unlock()

	time.Sleep(1 * time.Second)
}
```


场景：队列，发送方，发数据满了，停止 。消费方，没有数据可消费了，等待。 如果数据又写入了，通知所有消费方的协程都醒来。