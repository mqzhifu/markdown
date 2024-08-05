# linkerd调研结果

## 负载均衡

linkerd 公共参数

maxEffort：负载均衡器重试多少次后将节点标记为不可用

decayTimeMs

smoothWin:

highLoad

lowLoad

minAperture

需要考虑水平扩展

roundRobin:轮询

优点：无状态，简单

缺点：不能动态计算某些机器的负载

测试\-散列度

100以内还好

1000后，出现：

Exceeded 10.seconds binding timeout while connecting to /\#/io.l5d.consul/dc\-test01/t1\_test\_by\_wdy for name: /svc/t1\_test\_by\_wdy

Unable to establish connection to 172.19.149.29:80. service name: /svc/t1\_test\_by\_wdy client name: /\#/io.l5d.consul/dc\-test01/t1\_test\_by\_wdy addresses: \[172.19.149.29:80, 172.19.149.29:81\] selected address: 172.19.149.29:80

串行 并行 的结果差不多，没做到100%轮询，有一定误差

测试\-延迟

一个端口正常 81 ，另一个端口sleep 10秒 80

串行请求：一半机率落到 81,也就是，一个非常快，接着另一个非常慢

并行请求：80端口21个请求，81端口79个请求。这么看，LINKERD的配置，也就是1次重试失败就标记不可用，生效了，但是依然还是有21个请求打过去了。

p2c emwa 都是：随机\+带权，随机挑2台机器，然后统计2台机器的未完成请求数，哪个未完成请求数少，就代表负载小，且会把请求打到这台机器上。emwa是在这个基础之上，把统计的未完成请求数再加个权重，就是过去N秒内，2台机器的平均响应时间。说的再直白点：决定指向哪台机器要参考两个值：未完成请求数\+过去N秒内的响应时间

Power of Two Choices \(P2C\): 最小负载

随机找2个副本，并选择，负载较少的。负载由：每个副本的未完成数决定

缺点：1虽然统计了成功次数，但是有些服务器虽然成功了，但是响应时间过长。

2需要额外统计成功次数

测试结果：还会往延迟的机器打请求，在8%~50%左右。如果上来直接开若干请求，最高点还会往延迟的机器上打到50%，然后，再跑一次，会降到30%，然后就是20%，最后就徘徊在8%~20%左右

Power of Two Choices：峰值EWMA

随机找2个副本，并选择，但并不是统计未完成数，而是统计某个副本的延迟平均时间.

缺点：需要额外统计每次请求的耗时，再计算平均值

测试结果：1% ~ 42% ，还是第一次请求，出现42%左右进入负载高的机器上。但是越往后值越低，最后能降到1%。

P2C 和 EWMA ，跟并发数有一定关系，如果第一次并发来的值小一些，他请求负载高就小一些，因为自身的统计系统已生效，如果第一次并发数极高，完全可能把负载高的机器压死。

Heap: 堆负载

将每个副本的请求完成数，记录在小根堆中。

优点：不随机，统一观察所有副本，比较公平。

缺点：因为是统一观察，所以，是一块共享内存，竞争比较大

测试结果：第一次，直接50%....... 2%~50% ，但2%之前出现了一次异常。感觉跟P2C有点像，但是稳定性、负载高的机器请求率下降，不太友好。

ChannelException at remote address: 172.19.149.29/172.19.149.29:81 from service: \#/io.l5d.consul/dc\-test01/t1\_test\_by\_wdy. Remote Info: Upstream Address: Not Available, Upstream id: Not Available, Downstream Address: /172.19.149.29:81, D

ownstream label: \#/io.l5d.consul/dc\-test01/t1\_test\_by\_wdy, Trace Id: 328443312cba1dba.d4fe37d4bff516e4\<:328443312cba1dba

com.twitter.finagle.ChannelClosedException: null at remote address: 172.19.149.29/172.19.149.29:81. Remote Info: Not Available from service: \#/io.l5d.consul/dc\-test01/t1\_test\_by\_wdy. Remote Info: Upstream Address: Not Available, Upstream

id: Not Available, Downstream Address: /172.19.149.29:81, Downstream label: \#/io.l5d.consul/dc\-test01/t1\_test\_by\_wdy, Trace Id: addb7e82dc666d8d.326a7af7d9f57d0b\<:addb7e82dc666d8d

com.twitter.finagle.ChannelClosedException: null at remote address: 172.19.149.29/172.19.149.29:81. Remote Info: Not Available from service: \#/io.l5d.consul/dc\-test01/t1\_test\_by\_wdy. Remote Info: Upstream Address: Not Available, Upstream

id: Not Available, Downstream Address: /172.19.149.29:81, Downstream label: \#/io.l5d.consul/dc\-test01/t1\_test\_by\_wdy, Trace Id: 250fbf4d176f660d.5f9911d862649596\<:250fbf4d176f660d

测试挂了：

如果服务的所有IP 全挂，50%\-50%

如果，打开其中一个，所有请求转向这个正常的

不过，隔一段时间，再请求，会再打到 挂的那个，1\-2个

maxEffort：当LB，选择了一台负载较小的机器A后，开始建立连接请求，但是失败（比如：机器A突然触发了熔断，造成该机器被标识为不可用）。这个时候，要重新再试一次，试过N次后，该机器被标记为不可用。同时用上一次成功的机器，如果所有机器都失败，会随便挑一个。

## 熔断

熔断器/断路器：Fail Fast \- 会话\(连接\)驱动的断路器、Failure Accrual \- 请求驱动的断路器

快速熔断：连接阶段，也就是TCP连接。连接失败，直接就做处理，不等返回状态值。

如果在连接过程中，出现失败，直接把该IP副本，移到后台，尝试重新连接。

优点：在连接层面就介入，快速响应，直接把坏掉的副本移除

缺点：如果后端副本只有一个，那么，就是灾难了。。。所有请求都失败。

累计熔断：请求阶段（以响应值为准），TCP连接OK，查看返回信息。比如：500，就算是一次失败。

如果连续，接收N次失败，比如500。将标记该副本为不可用，通过参数failureAccrual，尝 试不可用的机器是否已经恢复。因为是靠响应值，所以可以使用responseClassifier 参数

以上的具体配置由Failure Accrual参数管理

Failure Accrual

kind: io.l5d.consecutiveFailures

failures: 5

kind : io.l5d.consecutiveFailures|io.l5d.successRate|io.l5d.successRateWindowed|none.

Consecutive Failures:观察每个节点连续失败的数量,当超过设定阈值后，实施退避停止向节点发送请求，连续失败次数由参数failures 控制

routers:

- protocol: http
    client:
    failureAccrual:
    kind: io.l5d.consecutiveFailures
    failures: 5
    backoff:
    kind: constant
    ms: 10000

Success Rate:计算每个节点指数加权的动态平均成功率，当成功率低于设置值时实施退避。计算成功率的样本窗口大小是固定的请求数量，由参数requests设置。

successRate:请求\-成功率

requests:计算成功率的请求样本数量

- protocol: http
    client:
    failureAccrual:
    kind: io.l5d.successRate
    successRate: 0.9
    requests: 20
    backoff:
    kind: constant
    ms: 10000

Success Rate \(windowed\):计算每个节点指数加权的动态平均成功率，当成功率低于设置值时实施退避。计算成功率的样本窗口大小是固定的时间间隔

successRate:请求\-成功率

window:计算成功率的请求时间间隔

None:关闭该功能

routers:

- protocol: http
    client:
    failureAccrual:
    kind: none

一但副本被标记为不可用，请求就不会再打到这个上面，那就得有恢复机制。

backOff 参数就管这个 （退避策略，配置重新发送请求的间隔策略，等待多久后重新请求node）

两种 类型：1、恒定 ，2 抖动（递增时间，直到最大值）

ms：每个请求的重试间隔

minMsrequired每个请求重试的最小间隔时间，单位毫秒

maxMsrequired每个请求重试的最大间隔时间，单位毫秒

对，影响值的标注，也就是哪些影响码标记为正常，哪些影响码标记为失败。

\#=====================熔断 END

cat /tmp/test\_linkerd\_lb/test\_rs/p2c/1.log |awk '{print $1}'|sort |uniq \-c

cd /data/ops/ops/linkerd/docker

./restart.sh

docker ps |head \-2 |awk '{print $1}'|tail \-1

docker exec \-it `docker ps |head -2 |awk '{print $1}'|tail -1` /bin/bash

docker inspect \-f "{{.Mounts}}" 2047c9901bcb

docker stats \-\-no\-stream `docker ps |head -2 |awk '{print $1}'|tail -1`|tail \-1|awk '{print $3,$7}'

============debug=================

./linkerd \-log.level=ALL \-log.output=/aaa/bbb/a.log

http://39.107.127.244:9990/admin/metrics.json
