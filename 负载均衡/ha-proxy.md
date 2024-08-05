
主要功能：

负载均衡：将前端进入的请求，平均分散到后端的若干台服务器上(L4和L7两种模式)

健康检查：支持TCP和HTTP两种健康检查模式(监听端口 /请求URI/请求获取头部信息)

会话保持：对于未实现会话共享的应用集群，可通过 Insert Cookie/Rewrite Cookie/Prefix Cookie，以及上述的多种 Hash 方式实现会话保持

SSL：HAProxy 可以解析 HTTPS 协议，并能够将请求解密为 HTTP 后向后端传输
HTTP 请求重写与重定向

监控与统计：HAProxy 提供了基于 Web 的统计信息页面，展现健康状态和流量数据。基于此功能，使用者可以开发监控程序来监控 HAProxy 的状态



|                   |                                     |      |     |     |
| :---------------- | :---------------------------------- | :--- | :-- | --- |
| RoundRobin        | 轮询 按照服务器ID顺序分发请求 权重                 | 短    |     |     |
| Static-RoundRobin | 轮询(静态)  跟上面一样，但修改权重无效 ，需要重启         | 短    |     |     |
| LeastConn         | 选择含有最少连接数的最近最少使用的服务器                | 长    |     |     |
| source            | 源ip地址HASH，每个IP请求固定服务器               |      |     |     |
| uri               | 对URI或部分URI进行 hash ，相同URI 永远访问一台服务器  | HTTP |     |     |
| hdr               | 依据指定的 HTTP heder 内容来选择对应服务器         |      |     |     |
| rdp-cookie        | 根据请求头中的 cookie hash，同一个用户，必定访问一个服务器 |      |     |     |
| url_param         | 根据请求参数/post 数据进行 hash               |      |     |     |
| random            | 根据随机数，请求到某一台服务器                     |      |     |     |
|                   |                                     |      |     |     |



