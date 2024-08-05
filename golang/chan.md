# 流转图

![[h_chan.png]]

# 源码

> go/src/runtime/chan.go

```go
type hchan struct {
	qcount   uint           // 循环数组，中的元素总量
	dataqsiz uint           // 循环数组，缓冲大小，=make(chan T, x)中的x
	buf      unsafe.Pointer // 缓冲区地址,是个循环数组
	elemsize uint16         // 元素大小，单位为字节
	closed   uint32         //chan关闭标记
	elemtype *_type         // 元素类型
	sendx    uint   // 待发送元素在缓冲器中的索引
	recvx    uint   // 待接收元素在缓冲器中的索引
	recvq    waitq  // 接收等待队列，用于阻塞接收协程
	sendq    waitq  // 发送等待队列，用于阻塞发送协程
	
	lock mutex  //自旋锁
}
```


|          |                               |     |
| :------- | :---------------------------- | :-- |
| lock     | 互斥锁，保证线程安全                    |     |
| buf      | 存数据的容器（循环数组结构）                |     |
| elemsize | 每个-元素大小，单位为字节                 |     |
| elemtype | 元素类型，用于数据传递过程中的赋值             |     |
| dataqsiz | 循环数组真实大小 ，make(chan T, x)中的 x |     |
| qcount   | buf 中已存储的数据个数                 |     |
| recvq    | 哪些G在等待接收(记录缓存中的数据读取到哪里了)      |     |
| sendq    | 哪些G在等待发送(记录缓存中的数据发送到哪里了)      |     |
| sendx    | buf队列，可以发送数据的索引值              |     |
| recvx    | buf队列，可以接收的数据的索引值             |     |
#### 分析：

- 所有的操作都是围绕 buf，用来给 协程 进行 IO 数据
- recvq、sendq 用来处理并发的 G：阻塞/唤醒 G，IO数据
- lock：保证并发安全


waitq：被阻塞的/等待的G，使用这个结构体，是个双向链表结构

```go
 type waitq struct {
     first *sudog
     last *sudog
 }
```


当写入方，发现队列满了后，创建一个 sudog：
1. 自己的 G 地址
2. 写不进入的数据的地址
3. chann 的地址
加入到 sendx 中，让出当前线程执行权。


# 写操作

1. 如果 recvq 队列  不为空，从 recvq 取出 一个 G ，并把数据写入，最后把该 G 唤醒
2. 如果缓冲区中有空余位置，则将数据写入缓冲区
3. 以上两条都不满足
	- 创建一个 sudog 结构体，并初始化成员变量
	- 把 数据 保存到 sudog.elem 中，以便等到 G 唤醒时，传递数据
	-  sudog 与 G 进行 关联
	- G 设置为 等待状态，且等待的是新创建的 sudog
	- 把 sudog 结构体扔到 等待队列中
	- gopark 阻塞当前协程



# 读操作

- 如果等待发送队列 sendq 不为空，且没有缓冲区，直接从 sendq 中取出 G ，把 G 中数据读出，最后把 G 唤醒，结束读取过程
- 如果等待发送队列 sendq 不为空，说明缓冲区已满，从缓冲区中首部读出数据，把 G 中数据写入缓冲区尾部，把 G 唤醒，结束读取过程
- 如果缓冲区中有数据，则从缓冲区取出数据，结束读取过程
- 将当前 goroutine 加入 recvq ，进入睡眠，等待被写 goroutine 唤醒



# 关闭操作


closed ：先是打上一个标识，但管道里的值不做清空，释放所有 recvq 等待队列，并存入 glist 等待唤醒 ，再释放所有 sendq，同样存入 golist，等待唤醒 ，最后唤醒所有glist里的协程。


>一个 channel  只能关闭一次，否则 panic


#### 向一个已关闭的 chan 写入数据

会报 panic

如果此时要写入数据的 G 在等待队列，chann 被关闭了，它会被唤醒，但是此时 chan 已经被关闭了，那么会报 panic

#### 向一个已关闭的 chan 读数据

不会报 panic ，如果 buf 里有数据还是会读出来，但当 buf 为空后，读取的数据是：buf 元素的默认值


如果是阻塞模式，一但被关闭就被告知不用阻塞了，唯一能做判定的是：读出的值如果是一个类型的默认值即为被关闭了，这个挺模糊的，但有其它方法：

```go
 if v, ok := <- ch; ok {
     fmt.Println(v)
 }
```


广播：如果只有一个G 进行了关闭操作，读取有多个，且是没有 buff 的 chan ，那么就是广播操作了，所有的读取G都能收到关闭通知。





# 唤醒条件：

- 当前 recvq 队列有等待的G
- 此时有 G ，进行写操作，上面的G会被唤醒

换个角度理解：它并没有所谓的调度器，来给等待队列上的G更新状态，而是由读写操作触发的

# 锁-线程安装

因为有 sendx  recvx 两个索引值，chan 的读写是可以并发的。但是，无法保证，同一时间可能有多个协程进行读操作，或者同一时间多个G进行写操作。

于是 ，LOCK 成员变量就是解决些事情的。


# Channel的缺点


#### 死锁

情况1：
```go
func main() { 
	ch := make(chan string) 
	ch <- "channelValue" 
}
```

只有一个协程，创建了个管道，不管你是读还是写，没有其它协程来读写数据，那就是死锁了

情况2：
```go
func main() { 
	ch1 := make(chan string) 
	ch2 := make(chan string) 
	go func() { 
		ch2 <- "ch2 value" 
		ch1 <- "ch1 value" 
	}() 
	<- ch1 
}

```

main 执行到 <- ch1  会进入阻塞
此时，子协程开始执行，但是在  ch2 <- "ch2 value" 阻塞了，导致  ch1 <- "ch1 value" 没有执行，死锁

情况3：
```go
func main() { 
	chs := make(chan string, 2) 
	chs <- "first" 
	chs <- "second" 
	
	for ch := range chs { 
		fmt.Println(ch) 
	}
}
```
range chs 会把数据读完，之后进入阻塞，但是因为在主协程中，跟情况1类似，死锁

情况4：

```go
func main() { 
	ch := make(chan int) 
	num := <-ch 
	fmt.Println("num=", num) }
```
此时的 chan 仅仅是创建了，实际它是个 nil ，读取 nil 会阻塞，且在 main 函数中，单协程模式，会造成死锁



情况5：
```go
func main() { 
	ch1 := make(chan string) 
	ch2 := make(chan string) 
	go func() { 
		ch2 <- "ch2 value" 
		ch1 <- "ch1 value" 
	}() 
	go func() { 
		<- ch1 
		<- ch2 
	}() 
	time.Sleep(time.Second * 2) }

```
有趣是，这种情况就不会死锁，原因是：主协程没有被阻塞。

总结：
- 定义了 chan ，就要正常用起来，不要只有写没有读
- 不要在 main 主协程中做 chan 操作
- 一个函数内或一个协程内，进行读写



####  数据拷贝

channel 中传递的都是数据的拷贝，可能会影响性能


#### 指针竞

channel 中传递指针会导致数据竞态问题（data race/ race conditions）




# 这里有4个函数，就不上源码了

1. makechan:创建一个chan
2. chansend1:给一个chan发送一条消息
3. chanrecv1:从一个chan接收消息
4. closechan:关闭一个管道