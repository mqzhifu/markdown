

# MPM

Multi-Processing Modules，处理并发连接和请求的模块


![[apache-mpm.png]]


prefork：启动之初，预派生一些子进程，用于接收请求/处理请求
worker：主进程+子进程+(子线程+侦听线程)
event：没看懂网上说的，只知道说 有专门的线程来管理 keep-avlie ，其余的跟 worker 一样

>网上资源太少，反正，我总结：多进程+多线程


从配置文件中看：
```

StartServers               4   # 默认启动进程数
MinSpareThreads         16   # 最小线程
MaxSpareThreads         512 # 最大线程
ThreadsPerChild           64      # 最大子线程数
ServerLimit                 32       # 进程最大数
MaxRequestWorkers     2048 # 最大请求数量
MaxConnectionsPerChild   10000 # 最大连接次数,超过后释放线程

```


网上的资源好少，自己日常使用中也没有太高深的东西。
个人感觉，现在都在用 NGINX ，APACHE HTTPD  越来越被忽视，甚至后一代的程序根本就不知道 这东西的存在。