# 引用包

查看稳定（使用的人多否、文档多否）
# 避免裸  go func 开协程

某一个函数pinic ，导致整个进程全挂
1. 要有 defer recover
2. 使用官方的 sync.WaitGroup
3. 单独封装一个方法，所有新开协程的调用 这个方法，由此方法统一新开协程

# 避免内存逃逸 

go build -gcflags="-m -l" main.go

1. 函数返回值(局部变量)：不要使用引用/地址
2. 闭包函数，引用外部变量
3. 参数中：空接口，interface{}
4. 一个变量过大
5. 指针被存入 slice、map、channel


# defer 关闭FD

像：文件、连接等，这些资源类的，打开后，函数结束后，尽量给关掉
1. 打开的文件
2. 打开的TCP连接（DB、redis ）
3. 锁


# defer try catch 捕获异常

可能会出现问题的代码，尽量都捕获一下

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