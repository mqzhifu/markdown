# zipkin

一次trace由一组span组成

traceId:一个请求的，完整链路ID.

spanId:经过的每个节点，都会产生这个ID

parentId:spanId的父ID

collector:收集器

storage：存储器

memory mysql es

search：查找

webui：可视化管理工具

Annotation

基于一个sapn，的上下文信息

event type :

cs：Client Start,表示客户端发起请求 ；一个span的开始；

cr：Client Received,表示客户端获取到服务端返回信息；一个span的结束

```
 sr：Server Receive,表示服务端收到请求
 ss：Server Send,表示服务端完成处理，并将结果发送给客户端

sr-cs：网络延迟
ss-sr：逻辑处理时间
cr-cs：整个流程时间
```

简单理解：一个span是一次子请求，该请求的event type 就是详细描述的信息，且存于Annotation
