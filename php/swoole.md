
# 支持网络协议：

| 协议                      | 函数                           |
| :---------------------- | :--------------------------- |
| TCP                     | Connect Receive Close Send   |
| UDP                     | Packet                       |
| UnixSocket Stream/Dgram |                              |
| Web-socket              | Open Message Disconnect Push |
| HTTP/HTTPS/HTTP2        | Request                      |
| SSL/TLS                 |                              |
| MQTT                    | Connect Receive Close        |

# 进程/线程组


![[swoole-进程组.png]]

启动过程（master 进程 ：生成 2+N+M个进程）：
- MASTER + MANAGER
- N为WORKER
- M为TASK

master 进程：set 配置信息 创建 manager 进程，创建 reactor 线程组
manager 进程：创建/管理 worker 、task 进程组
work 进程组：真正处理 业务逻辑 的进程，同时管理 定时器任务
task 进程组：woker 可以向 task 投递异步任务，然后回调 worker

> worker 进程组 、 reactor 线程组 的数量，主要取决于CPU个数


master 进程组 和 线程组 的实际工作内容：
- master 主线程： accept + 信号处理等 + 定时任务 
- reactor 线程组：TCP、网络IO、心跳检测线程等
- 心跳检测线程
- UDP收发包线程

reactor 线程组：
- C 端连接 ，S端 accept 拿到一个新FD
- 分从 reactor 线程组里，找一个，并分发给它
- 主要功能如：receive、sendto、close、buffer、dispatch
- 数据的具体处理，分再找一个 worker 进程处理（IPC 管道）
- 使用了：epoll 模式，跟NGINX 略像


![[php_swoole_process_structure.png]]
# reactor  dispatch_mode

reactor 是主要负责网络 IO的，它会发给给后面的 worker 进程。都是以组为单位，得有分发策略


轮循模式 ：随机分配。比较均匀

固定模式：根据连接的文件描述符分配 worker。这样可以保证同一个连接发来的数据只会被同一个worker处理

抢占模式：主进程会根据Worker的忙闲状态选择投递，只会投递给处于闲置状态的Worker

IP分配：根据客户端IP进行取模hash，分配给一个固定的worker进程。可以保证同一个来源IP的连接数据总会被分配到同一个worker进程 。ip2long\(ClientIP\) % worker\_num

UID分配：需要用户代码中调用 $serv\-\> bind\(\) 将一个连接绑定1个uid。然后swoole根据UID的值分配到不同的worker进程。算法为 UID % worker\_num，如果需要使用字符串作为UID，可以使用crc32\(UID\_STRING\)

stream 模式，空闲的worker会accept连接，并接受reactor的新请求

# Worker 进程的生命周期：


当一个Worker进程被成功创建后，会调用 onWorkerStart 回调，随后进入事件循环等待数据。当通过回调函数接收到数据后，开始处理数据。如果处理数据过程中出现严重错误导致进程退出，或者 Worker进程处理的总请求数达到指定上限，则Worker进程调用onWorkerStop回调并结束进程。

Worker进程通过 Unix Sock 管道将数据发送给 Task Worker


# task

```php
$serv = new Swoole\Server('127.0.0.1', 9501);

//设置异步任务的工作进程数量。
$serv->set([
    'task_worker_num' => 4
]);

//此回调函数在worker进程中执行。
$serv->on('Receive', function($serv, $fd, $reactor_id, $data) {
    //投递异步任务
    $task_id = $serv->task($data);
    echo "Dispatch AsyncTask: id={$task_id}\n";
});

//处理异步任务(此回调函数在task进程中执行)。
$serv->on('Task', function ($serv, $task_id, $reactor_id, $data) {
    echo "New AsyncTask[id={$task_id}]".PHP_EOL;
    //返回任务执行的结果
    $serv->finish("{$data} -> OK");
});

//处理异步任务的结果(此回调函数在worker进程中执行)。
$serv->on('Finish', function ($serv, $task_id, $data) {
    echo "AsyncTask[{$task_id}] Finish: {$data}".PHP_EOL;
});

$serv->start();
```



# 关键函数


send：异步，向某个C 发送消息，可以发送 unixSocket
sendwait：同步，向某个C 发送消息
sendTo：向任意的客户端 IP:PORT 发送 UDP 数据包。
sendMessage：向任意 Worker 进程或者 Task 进程发送消息


# 信号 


SIGTERM: 向主进程/管理进程发送此信号服务器将安全终止。
SIGUSR1: 向主进程/管理进程发送SIGUSR1信号，将平稳地重启所有Worker进程。
SIGUSR2: 向主进程/管理进程发送SIGUSR2信号，将平稳地重启所有Task进程。
SIGKILL：强制杀死进程，类似于 kill -9 xxx 。适用于 强制停止/强制重启 服务。
