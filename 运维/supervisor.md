# supervisor

http://supervisord.org/plugins.html?highlight=cesi

## 安装

官方给的方法:

> pip install supervisor

我用的：它依赖了一个py3的包

> yum install supervisor

yum 挺方便，完成后，会有两个指令:

supervisord:守护进程/主进程

supervisorctl:管理 supervisord

etc目录下会有一个新文件和一个新的目录：

/etc/supervisord.conf:核心配置文件

/etc/supervisord.d:空目录

supervisord.conf 主配置文件

```
loglevel=info
logfile=/var/log/supervisor/supervisord.log 
pidfile=/var/run/supervisord.pid
serverurl=unix:///var/run/supervisor/supervisor.sock
[include]
files = supervisord.d/*.ini
```

新版unix\_http\_server必须得打开uname \+ ps

> 主配置文件中，会包含supervisord.d目录下的子配置文件

创建一个demo文件

> touch /etc/supervisord.d/demoone.ini

```
[program:demoone]   ;脚本名称
command=/www/accountServer/accountServer ;执行的启动指令
directory=/www/accountServer/   ;脚本工具目录
autorstart=true     ;supervisor启动后，自动启动该脚本
autorestart=true    ;脚本进程挂了后，自动拉起
startsecs=3
startretries=3      ;脚本进程挂了后，自动拉起重试次数
stdout_logfile=/www/server/panel/plugin/supervisor/log/login.out.log
stderr_logfile=/www/server/panel/plugin/supervisor/log/login.err.log
stdout_logfile_maxbytes=2MB
stderr_logfile_maxbytes=2MB
user=root
priority=999
numprocs=1
process_name=%(program_name)s_%(process_num)02d

```

启动

> supervisord \-c /etc/supervisord.conf

进程定义名：可能出现两部分，如果DIY配置文件中的process\_name，也就是不等于默认%\(program\_name\)s,再简单点进程名与demoone不一样，即两分钟：demoone:\[diy process\_name\]

## 日常操作指令

\#管理supervisor中的应用进程

supervisorctl status 进程定义名

supervisorctl stop 进程定义名

supervisorctl restart 进程定义名

\#管理supervisord守护进程

supervisorctl reread

supervisorctl reload

supervisorctl update

supervisorctl shutdown

## web\-ui

自带的：

```
[inet_http_server]      ; 
port=0.0.0.0:5555       ; 
username=admin           ; 
password=123456          ;
```

丑是丑了点，但核心功能都有...基本上能覆盖主要的命令行指令,官网还提供了几个UI的:

Nodervisor:nodejs，URL地址都是错的...

Django\-Dashvisor:py,2015年停更

SupervisorUI:php

Supvisors:py

supervisorui:php 2012年就停更了

multivisor:py

cesi:py

Supervisord\-Monitor:php，2017年3月停更

算下来就两个还靠点谱吧：Supervisord\-Monitor 和cesi

cesi是比较好看的，无奈PY依赖包安装不上，放弃了...

官方自带的只能监控单机节点，3方包可监控多节点
