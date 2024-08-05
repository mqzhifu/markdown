
# 概览

#### 传统模型

![[golang-传统线程模型.png]]

传统模型：开发者调用系统的 API，创建线程，线程最终在 CPU 执行。
优点：开发者比较简单
缺点：OS 调试 线程 比较吃性能

#### golang 模型

![[golang-线程模型.png]]

golang 把线程的调度写在了用户态
优点：性能更好
缺点：部分情况，线程的调度不如OS

# 术语


|      |                  |                                |     |     |
| :--- | :--------------- | :----------------------------- | :-- | --- |
| M    | machine          | 真实的一个线程，执行一个 goroutine 挂载的函数代码 |     |     |
| P    | process          | 介于GM之间，调度G和M                   |     |     |
| G    | goroutine        | 一个协程。挂载一个执行函数                  |     |     |
| allP | process list     | 存储所有的P容器                       |     |     |
| allM | machine list     | 存储所有的M容器                       |     |     |
| LRQ  | Local Run Queue  | 包含G的队列，每个P有一个                  |     |     |
| GRQ  | Global Run Queue | 包含G的队列，特殊情况会进到这里               |     |     |
#### runtime

包含了调度、内存管理、垃圾回收、内部数据结构、定时器和各种系统调用的封装等。
>可以说golang的强大都归功于 runtime 的实现。
>GMP的核心也是  runtime

# 启动过程

![[golang-启动过程.png]]

指令行，启动 go 程序：
1. shell 会创建一个进程
2. 进程会创建一个线程(主线程 m0)
3. 之后的代码在主 m0 上开始执行


```C
CALL runtime·args(SB)
CALL runtime·osinit(SB)
CALL runtime·schedinit(SB)
//2.2 调用 runtime·newproc 创建一个协程
// 并将 runtime.main 函数作为入口
MOVQ $runtime·mainPC(SB), AX // entry
PUSHQ AX
CALL runtime·newproc(SB)
POPQ AX
//2.3 启动线程，启动调度系统
CALL runtime·mstart(SB)
```

1. runtime·args：初始化 执行文件的绝对路径 命令行参数
2. osinit：CPU 个数  、内存页大小
3. schedinit：
	- worldStopped：先停止GC
	- stackinit：初始化栈
	- mallocinit：初始化堆
	- mcommoninit：设定M个数 获取当前G 绑定M和G  设定 allM
	- gcinit：初始化垃圾回收
	- procresize：创建 P 列表，对新 P 进行内存分配和初始化
	- worldStarted：恢复GC
4. newproc：
	- 创建一个G，绑定 runtime·main 
	- 把 G 插入到 P 的 runQueue 中
	- wakeP
5. MStart:
	- 创建一个G，创建一个线程
	- 把G加入到 allG中
	- schedule
6. schedule
	- 从P中找一个G,用来执行 main goroutine
	- 死循环
7. main  goroutine
	- 创建一个 M，执行 sysmon
	- 创建一个 G，执行 gc_nable
	- 调用 用户的 main 函数
8. 程序在 schedule（findrunnable） 进入了死循环 
9. main  goroutine 执行体，算是最初的执行体，此函数被压到在栈的最底部
>这就是 main 一但结束了，其它 协程全结束的原因


>自旋：如果M在 本地队列/全局运行队列/netpoller中，找不到工作  
(M进入了一种循环找可运行G的状态)

# schedule

核心函数：schedule，用于调试 P M G：
1. 是否正在 GC
3. 当前 M 是否已经绑定了一个待执行的 G
4. 每61次后，尝试从 global  run queue
5. 尝试从 local run queue 拿 一个G执行
6. 从 findrunnable 找一个 G 来执行


 findrunnable 函数，深度的去寻找一个待执行的 G ：
- 尝试从 local run queue 拿 一个 G 执行
- 尝试从 global  run queue
- 从网络 - EPOLL 中找到可IO事件的G->更新状态为可运行
- 其它的 P 的 LRQ 是否有 G 等待执行
- 是否可以 GC
- 再次尝试 从 GRQ 中找G
- P 解绑当前M，把P 放到空闲列表中
- .......
- 睡眠

大概的整体流程分析：
1. 尽可能的从 LRQ 或 GRQ 队列中找一个待执行的 G
2. 从其它P的LRQ中偷取一些G
3. 执行GC
4. 把一些 sys_call 阻塞的 G ，更新一下状态。(算是把可运行的 G 移到 LRQ中)
5. 如果M被占用 ，解绑 P
6. 如果以上均未拿到G，把P归还到 空间列表，进行睡眠

>findrunnable 才是核心函数，它主要是找寻可执行的 G
# sysmon

每新建一个线程，就会连带着创建一个 sysmon 线程

会调用 retake() 函数，retake() 函数会遍历所有的P，如果一个P处于执行状态， 且已经连续执行了较长时间，就会被抢占


- 由应用程序创建的计时器（timers）。sysmon 查看应该在运行但仍在等待的计时器
>在这种情况下，Go将查看空闲的M和P列表以尽快运行它们。
- 网络轮训器（net poller ）和系统调用（ system calls）。它运行网络操作中阻塞的goroutines。
- 如果垃圾收集器，已经很长时间没有运行（超过2分钟），sysmon将强制一轮垃圾回收。
- 长时间运行 goroutine 的抢占（超过**10**毫秒）

**重要数字：**
- 一个G执行超过 10 毫秒 
- 2分钟未 GC 过
- sysmon 的循环周期时间：2US(微秒)~40MS(毫秒)

# GPM

![[golang-gmp.png]]


GPM 结构的体:


```C

type g struct {
	stack stack // 描述了真实的栈内存，包括上下界
	m *m // 当前的m
	sched gobuf // goroutine切换时，用于保存g的上下文
// 用于传递参数，睡眠时其他goroutine可以设置param，唤醒时该goroutine可以获取
	param unsafe.Pointer 
	atomicstatus uint32
	stackLock uint32
	goid int64 // goroutine的ID
	waitsince int64 // g被阻塞的大体时间
	lockedm *m // G被锁定只在这个m上运行
}

```

```c

type m struct {

	g0 *g // 带有调度栈的goroutine
	gsignal *g // 处理信号的goroutine
	tls [6]uintptr // thread-local storage
	mstartfn func()
	curg *g // 当前运行的goroutine
	caughtsig guintptr
	p puintptr // 关联p和执行的go代码
	nextp puintptr
	id int32
	mallocing int32 // 状态
	spinning bool // m是否out of work
	blocked bool // m是否被阻塞
	inwb bool // m是否在执行写屏蔽
	printlock int8
	incgo bool // m在执行cgo吗
	fastrand uint32
	ncgocall uint64 // cgo调用的总数
	ncgo int32 // 当前cgo调用的数目
	park note
	alllink *m // 用于链接allm
	schedlink muintptr
	mcache *mcache // 当前m的内存缓存
	lockedg *g // 锁定g在当前m上执行，而不会切换到其他m
	createstack [32]uintptr // thread创建的栈
}

```

```c
	type p struct {
	lock mutex	
	id int32	
	status uint32 // 状态，可以为pidle/prunning/...	
	link puintptr	
	schedtick uint32 // 每调度一次加1	
	syscalltick uint32 // 每一次系统调用加1	
	sysmontick sysmontick	
	m muintptr // 回链到关联的m	
	mcache *mcache	
	racectx uintptr	
	goidcache uint64 // goroutine的ID的缓存	
	goidcacheend uint64	
	// 可运行的goroutine的队列	
	runqhead uint32	
	runqtail uint32	
	runq [256]guintptr	
	runnext guintptr // 下一个运行的g	
	sudogcache []*sudog	
	sudogbuf [128]*sudog	
	palloc persistentAlloc // per-P to avoid mutex	
	pad [sys.CacheLineSize]byte
}

````


P绑定M，M绑定G，P是控制者
P决定M上应该执行哪个G：
- 可能是从LRQ 
- 也可能是GRQ
- 也可能是 chanel wait 里的G
- 也可能是 netPoll 的G
- ......

一但发生阻塞，P就与M解绑，让M自己玩儿去吧，P再重新找新的G，再找M，如果空闲列表里有就直接拿，如果没有就创建一个
>这里看出  P是跟CPU对应的，但是M可能会有多个

#### GPM 状态流转

#### G状态：

空闲中
待运行
运行中
系统调用中
等待中
已中止
栈复制中

#### M状态
>实际上它并没有这些状态，但如下算是虚拟的

自旋中
执行Go代码中
执行原生代码中
休眠中

#### P状态

空闲中
运行中
系统调用中
GC 停止中
已中止