# 概览

CGI全称"通用网关接口"（Common Gateway Interface），用于HTTP服务器与其它机器上的程序服务通信交流的一种工具，CGI程序须运行在网络服务器上。

FastCGI是一个可伸缩地、高速地在HTTP服务器和动态脚本语言间通信的接口（FastCGI接口在Linux下是socket（可以是文件socket，也可以是ip socket）），主要优点是把动态语言和HTTP服务器分离开来。多数流行的HTTP服务器都支持FastCGI，包括Apache、Nginx和lightpd。

web-service 的核心是：
1. 提供HTTP协议的通信
2. 管理静态文件


但，现实的场景是，静态文件太简单，有些复杂的需求，得有动态的输出结果，像：
1. JSP -> java
2. ASP .NET -> C#
3. PHP 

都能根据不同的请求，输出不的结果。

这就变成了：web-service + 处理动态请求的语言
也可以理解为： web-service +  FastCgi

这里只是做比喻，不太严谨。   JAVA因为必须得有JVM，他自己有TOMCAT  。 .net 有自己的IIS 。只有 PHP 在使用FastCGI


![[fastCgi.png]]

>没什么太复杂的，就是把 webservice 与 动态语言 分离，使用了 fastCgi 协议罢了


# PHP-FPM

实现了 fastCgi 协议

Master/Worker进程模型，master 主要是：
1. 接收 管理员相关信号
2. 管理 worker 进程

worker进程：监听端口，处理具体的业务逻辑



动态（Dynamic）：，PHP-FPM启动时会创建一定数量的Worker进程。当请求数逐渐增大时，会动态增加Worker进程的数量；当请求数降下来时，会销毁刚才动态创建出来的Worker进程。


静态（Static）：**PHP-FPM**启动时会创建配置文件中指定数量的Worker进程，不会根据请求数量的多少而增加减少


按需（Ondemand）：PHP-FPM启动时，不会创建Worker进程，当请求到达的时候Master进程才会fork出子进程



平时用动态多一点，它比较有弹性，可伸缩。但如果量特别大的话， 静态模式更稳定一些。
按需模式很少用。


感觉也没什么好说的，这东西火，是因为NGINX火了，PHP又原生给支持了。
理论上是：把 WEB-SERVICE 和 PHP 做了分离，偶尔能用上，但一般开发，还是会把 NGINX+FRM 部署在同一台机器上

