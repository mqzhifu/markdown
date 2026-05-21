
# 引用包

查看稳定（使用的人多否、文档多否）
# 避免裸  go func 开协程

# 避免内存逃逸 

# 内存泄露

# defer 关闭FD

# defer try catch 捕获异常

# 并发安全

同一个变量，被多协程同时IO

# 测试用例 

go test

覆盖率 go test -cover

# 基准测试 

go test -benchmem -bench .

# 查看火焰图 

pprof

# 变量 锁 竞争  race

go run -race main.go

# 如何做 Go 性能排查？


高并发 TCP 服务如何设计？协程模型、连接池、心跳检测？
帧同步架构设计？如何保证帧顺序、无锁管理玩家？
帧同步两种架构：状态同步 vs 帧同步区别、优缺点？
帧同步中 map 玩家列表如何做到无锁安全？Actor 模型落地？
UDP 丢包、乱序怎么处理？帧号时序如何严格保证？
TCP 粘包原因？常用分包协议设计方案？


# 无锁MAP/无锁队列 

无锁MAP： sync.map  免费算是无锁的，读的时候是特定一个数组，写的时候，只有当KEY不存在的时候才加锁。

无锁QUEUE:

```
func (q *LockFreeQueue) Push(val any) bool {
	for {
		tail := atomic.LoadUint64(&q.tail)
		head := atomic.LoadUint64(&q.head)

		// 💡 条件判断：如果队尾和队头差值达到了容量，说明队列满了
		if tail-head >= q.cap {
			return false // 满了，直接拒绝（可自定义阻塞或限流策略）
		}

		// 🚀 核心 CAS 遭遇战：尝试将全局队尾游标自增 1
		// 如果在这一瞬间别人没插足，自增成功，说明我【抢到了这个格子的专属写入权】！
		if atomic.CompareAndSwapUint64(&q.tail, tail, tail+1) {
			// 利用位运算替代传统的取模(%)，速度快了几十倍
			index := tail & (q.cap - 1)
			q.buffer[index] = val // 安全写入，绝无并发踩踏
			return true
		}
		

		// 🚀 核心 CAS：尝试抢占将队头游标自增 1
		if atomic.CompareAndSwapUint64(&q.head, head, head+1) {
			index := head & (q.cap - 1)
			val := q.buffer[index]
			q.buffer[index] = nil // 洗干净格子，防止内存泄漏/对象污染
			return val, true
		}

		
		// 抢格子失败，原地甩头（让出 CPU 周期）下一微秒继续重试（自旋）
		runtime.Gosched() 
	}
}
```
