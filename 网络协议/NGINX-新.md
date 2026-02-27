# 概览


#### 核心功能：

| 功能                | 例子                      |
| ----------------- | ----------------------- |
| 实现了 HTTP/HTTPS 协议 | 可以做文件服务器                |
| 实现了 WEBSOCKET 协议  | 浏览器也可以使用长连接             |
| 兼容 OS             | win mac linux           |
| 代理/反向代理           | 给后端做代理，可以加一层日志、权限控制、头信息 |
| 负载均衡              | 给后端一组服务器做负载均衡，并自带一些算法   |
| fastCgi           | PHP NODEJS              |

#### 其它功能：

| 功能           | 例子                                              |
| -------------- | ------------------------------------------------- |
| POP3/IMAP/SMTP | 邮件处理                                          |
| MEMCACHE       | 缓存处理                                          |
| LUA            | 写些脚本，动态挂载到 NGINX，控制 NGINX 的一些处理 |
| RTMP           | 视频流                                            |

它的核心功能其实就是处理 HTTP 请求。不用你再去写代码了。并且它的性能高、稳定强、可扩展。

# nginx 的核心优势

核心优点：

1. 性能高（epoll 处理发并发高）
2. 兼容主流平台( win mac linux )
3. 满足主流功能的需求(http 反向代理 )
4. 高度灵活，可模块化、可使用 LUA 嵌入开发、可用配置文件驱动

为什么性能高？

1. 小巧，并没有像 APACHE 融入太多功能。
> 原码文件一共就 6MB 左右\(未压缩的大小\)

1. 网络 IO 使用的是：EPOOL 模型
> epoll 为什么快，参考另外一篇文章

# nginx 模块

从 src 目录分析：

| 文件夹名   | 描述                             |
| ------ | ------------------------------ |
| core   | 核心模块，nginx 大部分的操作都在这里          |
| event  | 对 socketFD 的事件驱动处理             |
| http   | 对 http/https 协议的处事             |
| mail   | 对邮件的处理                         |
| misc   | 未知                             |
| os     | 网络、socketFD、文件 IO（各种 OS 的兼容处理） |
| stream | 各种数据流的处理，upstream              |

## core

| 模块                                       | 描述     |
| ---------------------------------------- | ------ |
| array list string buffer tree hash queue | 基础数据结构 |
| crc crypt md5 sha1                       | 加密算法   |
| cycle                                    | 生存周期   |
| inet                                     | 底层网络相关 |
| log                                      | 日志     |
| slab palloc                              | 内存处理   |
| proxy                                    | 代理     |
| regex                                    | 正则     |
| config_file                              | 配置文件解析 |
| epoll                                    |        |
| event                                    |        |
| events                                   |        |

## email

| 模块   | 描述 |
| ------ | ---- |
| pop3   |      |
| realip |      |
| smtp   |      |
| auth   |      |

## http

| 模块                  | 描述 |
| --------------------- | ---- |
| access                |      |
| addition_filter       |      |
| auth_basic            |      |
| auth_request          |      |
| autoindex /index      |      |
| chatset_filter        |      |
| fastcgi /scgi         |      |
| flv/ mp4              |      |
| geo/ geoip            |      |
| grpc                  |      |
| gunzip/gzip           |      |
| header_filter         |      |
| image_filter          |      |
| limit_conn limit_req  |      |
| log                   |      |
| memcache              |      |
| mirror                |      |
| proxy                 |      |
| random_index          |      |
| realip                |      |
| referer               |      |
| rewrite               |      |
| static                |      |
| stub_status           |      |
| ssl                   |      |
| try_files             |      |
| upstreamuserid_filter |      |
| uwsgi                 |      |
| http2                 |      |

## ngx_core_module:

| 模块     | 描述                                        |
| ------ | ----------------------------------------- |
| epoll  | linux 事件驱动库                               |
| kqueue | unix linux 事件驱动库                          |
| select | 轮询 socketFD 操作库                           |
| core   | 对 socketFD 的基础操作，如：accept connect openssl |

rds\-json\-nginx

lua\-nginx

# 进程组

master + worker+ cache ( loader + manager )

master：主要管理 worker。如果某 worker 进程挂了，它会重新拉起

worker：处理下下游戏的网络请求。数量与 CPU 核数对应

> master 也会创建 socker 监听端口，但最终还是会把处理权交给 workder.

cache-loader:上下游请求的缓存、常用文件的缓存

cache\-manager:主要是清理一些过期的缓存文件

# NGINX 启动过程

这里主要是分析的 main 函数

1. ngx_get_options，主要用于解析命令行中的参数，
> 如：nginx \-s stop | start | restart

1. ngx_time_init，初始化并更新时间
> 如： 全局变量 ngx_cached_time

1. ngx_getpid，获取当前进程的 pid。
> 用于发送重启，关闭等信号命令。

1. ngx_log_init，初始化日志，并得到日志的文件句柄 ngx_log_file.fd
2. init_cycle：Nginx 的全局变量。
> 在内存池上创建一个默认大小 1024 的全局变量。

1. ngx_save_argv，保存 Nginx 命令行中的参数和变量,放到全局变量 ngx_argv
2. ngx_process_options，将 ngx_get_options 中获得这些参数取值赋值到 ngx_cycle 中。
> prefix, conf_prefix, conf_file, conf_param 等字段。

1. ngx_os_init：初始化系统相关变量，
> 如内存页面大小 ngx_pagesize,ngx_cacheline_size,最大连接数 ngx_max_sockets 等

1. ngx_crc32_table_init，初始化一致性 hash 表，主要作用是加快查询
2. ngx_add_inherited_sockets：主要是继承了 socket 的套接字。主要作用是热启动的时候需要平滑过渡
3. ngx_preinit_modules，主要是前置的初始化模块，对模块进行编号处理
4. ngx_init_cycle 方法，完成全局变量 cycle 的初始化
5. ngx_signal_process，如果有信号，则进入 ngx_signal_process 方法。
> 如： nginx -s stop ， 则处理 Nginx 的停止信号
6. ngx_get_conf，得到核心模块 ngx_core_conf_t 的配置文件指针
7. ngx_create_pidfile，创建 pid 文件。
> 例如：/usr/local/nginx\-1.4.7/nginx.pid

1. ngx_master_process_cycle，这函数里面开始真正创建多个 Nginx 的子进程。包括：
- 子进程创建
- 事件监听
- 各种模块运行等都会包含进去

这里看，其主要的就是：

1. 创建全局变量
2. 解析/处理配置文件
3. 信号的注册与监听
4. 模块的初始化与创建
5. worker 进程组的创建、网络的监听

分析：ngx_master_process_cycle

1. ngx_master_process_cycle：
   1. 主进程进行信号的监听和处理
   2. 创建子进程
   3. 开启死循环模式
      ......
2. ngx_start_worker_process
   1. 创建 N 个 wokrder 进程（N=配置文件中的数量）
   2. 拿到新进程的 PID
   3. 将 PID 返回给 master ，用于信号处理
3. ngx_spawn_process:就是 fork 个进程，拿个 PID 做个简单的容错处理....
4. ngx_worker_process_cycle：
   1. ngx_workder_process_init
   2. ngx_process_events_and_timer
   3. 监听信号
   4. epool 回调处理
   5. 开启死循环模式
5. ngx_workder_process_init:
   1. 环境变量、全局变量
   2. 设备进程相关信息
6. ngx_process_events_and_timer：把自己注册到 epool 中去

# 配置文件

```
根配置 {
    ......
    http配置 {
        upstream配置 {
            ......
        }
        ......
        server配置块{
            ......
            location 配置块{
                ......
            }
        }
    }
}
```

优先集/包含关系：

```mermaid
graph LR

根配置 -->|A1| http-配置
http-配置 --> server-配置块
server-配置块 --> upstream-配置
upstream-配置-->  location-配置块

```


- 根配置：这里主要是控制 nginx 相关，像： 主日志、epool、worker 进程数等。基本上日常不用动，如果要动也是对高并发的优化处理。
- http 配置：这里就对 HTTP 协议的整体配置，如：日志、最大连接数、数据压缩、限流等。跟上面差不多，很少改。
- server 配置块：每一个域名就有一个 server 块。这里面就是具体的请求处理的配置块了。如：对域名、HTTS、公共页面配置\(404/403/503\)等。
- location 配置块：这里就是具体到 URI/URL 、代理、跳转、URL 重写、添加 http 头/响应头、FAST\-CGI 等。

server 和 location 是日常程序员/运维 使用场景最多的。另外，server 跟 location 的一些配置是均可复用的，只是作用域不太一样。

# 跨域设置

add_header Access\-Control\-Allow\-Origin \*;
add_header Access\-Control\-Allow\-Headers X\-Requested\-With;
add_header Access\-Control\-Allow\-Methods GET,POST,OPTIONS;

# 代理

只需要使用一个指令：proxy_pass，即可。

跳转到 3 方网站：

```
location /baidu {
    proxy_pass http://www.baidu.com;
}
```

跳转到自己域内的一台服务器:

```
location /user {
    proxy_pass http://192.168.1.1:8080/user;
}
```

跳转到某一个服务器集合:backend，并做负载均衡

```
location /user {
    proxy_pass backent;
}

upstream backend {
    server 192.168.1.1;
    server 192.168.1.2;
    server 192.168.1.3;
    ......
}
```

其它的参数：

\#上游请求推出错误时 或 超时

proxy_next_upstream error timeout http_503 http_504 http_502;

\#上游 建立连接 超时时间

proxy_connect_timeout 500s;

\#上游 读取超时 时间

proxy_read_timeout 500s;

\#上游 发送超时 时间

proxy_send_timeout 500s;

\#请求上游戏时 在 HTTP 头部加上 host 信息

proxy_set_header Host $http_host;

\#请求上游戏时 在 HTTP 头部加上 IP 信息

proxy_set_header X\-Real\-IP $remote_addr;

\#请求上游戏时 在 HTTP 头部加上 客户端的信息 信息

proxy_set_header X\-Forwarded\-For $remote_addr;

nginx 的代理确实简单，就一个 proxy_pass 。剩下的管理员可任意配置使用。其实，代理的核心原理也不难，就是把客户端的请求信息保存，然后再发送一个完全一样的请求，发送给后端服务器。

## upstream

容器，定义一组 server

例如：定义一组叫 backend 的 server

```
upstream backend {
    server backend1.example.com weight=5;
    server 127.0.0.1:8080 max_fails=3 fail_timeout=30s;
    server unix:/tmp/backend3;
    server backup1.example.com:8080 backup;
}
```

# 负载均衡

简单说，它就依赖一个指令：upstream，在这个基础之上，再配置点信息就实现了，依然是很简单。

1. 轮询
   每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器 down 掉，能自动剔除。
   upstream backserver {
   server 192.168.0.14;
   server 192.168.0.15;
   }
2. 指定权重
   用于后端服务器性能不均的情况
```c++
   upstream backserver {
	   server 192.168.0.14 weight=10;
	   server 192.168.0.15 weight=10;
   } 
```

3. IP 绑定\-hash
   每个访客固定访问一个后端服务器，可以解决 session 的问题
4. fair
   按后端服务器的响应时间来分配请求，响应时间短的优先分配。
```c++
upstream backserver {
	server server1;
	server server2;
	fair;
} 
```

5. url_hash
   按访问 url 的 hash 结果来分配请求，使每个 url 定向到同一个后端服务器，后端服务器为缓存时比较有效。
```c++
upstream backserver {
	server squid1:3128;
	server squid2:3128;
	hash $request_uri;
	hash_method crc32;
}   
```


# 一次 HTTP 请求，主要的 11 个阶段

| 步骤                          | NGINX 的模块                               |
| ----------------------------- | ------------------------------------------ |
| NGX_HTTP_POST_READ_PHASE = 0  | realIp                                     |
| NGX_HTTP_SERVER_REWRITE_PHASE | rewrite                                    |
| NGX_HTTP_FIND_CONFIG_PHASE    |                                            |
| NGX_HTTP_REWRITE_PHASE        | rewrite                                    |
| NGX_HTTP_POST_REWRITE_PHASE   |                                            |
| NGX_HTTP_PREACCESS_PHASE      | limit_req \-\> limit_conn                  |
| NGX_HTTP_ACCESS_PHASE         |                                            |
| NGX_HTTP_POST_ACCESS_PHASE    | access auth_basic auth_request             |
| NGX_HTTP_PRECONTENT_PHASE     | try_file mirrors                           |
| NGX_HTTP_CONTENT_PHASE        | concat random_index index autoindex static |
| NGX_HTTP_LOG_PHASE            | access log                                 |

> 结构体：ngx_http_phases

具体分析 11 步骤：

1. NGX_HTTP_POST_READ_PHASE：
   拿取 HTTP 头信息，如：IP PORT 代理 IP ，并保存到变量中，如：remote_addr

> 这里还有个 real_ip 模块，可选项

1. NGX_HTTP_SERVER_REWRITE_PHASE：
   寻找 server 块中的 rewrite
   1. 404 403 502 error_page
   2. return
2. NGX_HTTP_FIND_CONFIG_PHASE
   寻找 location，nginx 会把每个 server 中的 每个 location 正则提取到一个二叉树中做索引，然后请求的 URI 在树中做搜索
3. NGX_HTTP_REWRITE_PHASE
   寻找 location 块中的 rewrite。这里可能会重复执行，因为 rewrite 跳转一次，再进来可能继续 rewrite ....
4. NGX_HTTP_POST_REWRITE_PHASE
   rewrite 后，为防止死循环
5. NGX_HTTP_PREACCESS_PHASE
   认证预处理，如：limit_conn  limit_req
6. NGX_HTTP_ACCESS_PHASE
   权限认证：
   1. IP 限制，如： deny 192.168.1.1; allow 192.168.1.0/24;
   2. auth_basic ：用户认证，浏览器自带的用户认证机制。
   3. auth_request ：权限认证，当有一个请求进来，NGINX 并不直接请求后端的服务器，而是先发一个 auth 请求，后端如果返回 200，则才会正常代理该请求到后端，否则直接拒绝
7. NGX_HTTP_POST_ACCESS_PHASE
   权限认证后置处理
8. NGX_HTTP_PRECONTENT_PHASE
   就一个 try_file 模块
   1. 可以给 location 找不到文件的时候，做个默认文件\(重定向\)
   2. 可以增加目录，用于寻找 URI 对应的静态文件
9. NGX_HTTP_CONTENT_PHASE
   响应内容
10. NGX_HTTP_LOG_PHASE
    日志

# 总结

nginx 或 apache ，做为 webservice 类型的开源软件，确实是方便。通过一个配置文件，用于驱动使用 webservice。程序员不需要再做额外的开发，就能使用 http 相关的服务。另外，nginx 还保证了稳定、性能、可扩展等等。真的是省事省人省力。

不过，过度使用开源免费的软件，也会拉低程序员的基础水平。对底层的理解不够深刻、培训机构的填鸭式教育陪着了一堆混子等等吧。但咋说呢，应该顺应时代发展。向前发展，有些底层的原理也不一定完全能理解。站在巨人的肩膀上！
