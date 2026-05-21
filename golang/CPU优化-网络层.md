

# CPU 缓存

CPU处理的速度   寄存器-》L1-》L2-》L3-》内存

如果你的 Go 代码引发了一次 Cache Miss（缓存未命中），CPU 必须越过 L1/L2/L3 直接去主内存里读数据，CPU 就会在这里白白干等 上百个时钟周期。



# Go netpoll 原理


如果当前协程，使用网络IO，且需要等待，被阻塞。
GO会把当前G和当前M解绑，把G扔到：pollDesc 等待唤醒队列，在epoll里注册这个FD的事件。


网卡收到了数据，通过中断通知了 Linux 内核，内核把这个 Socket 标记为了“可读”。

sysmon  GMP 调度 GC，这3种情况都会唤醒G。




# reactor/proactor

reactor：就是EPOLL模式，你监听哪些FD，这些FD ready 了，可IO了，唤醒你，你来自己自己数据

proactor：你直接给内核一个异步读指令，就不管了。等内核都读完了，走你的回调函数
得提前准备好一块内存给 proactor 写入
多核 CPU 的利用效率与“缓存命中

