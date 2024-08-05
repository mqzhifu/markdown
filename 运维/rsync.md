# rsync

## rsync

```
touch /soft/rsync/rsyncd.conf  
vim /soft/rsync/rsyncd.conf
```

```
port = 8877
log file = /data/logs/rsync/rsyncd.log
pid file = /var/rsyncd.pid
max connections = 5
strict modes = no

[install]
uid = rsync
gid = rsync
path = /install/zip
read only = no
write only = no   
hosts allow = *
hosts deny = 10.5.3.77
```

服务端\-启动

> rsync \-\-daemon \-\-config=/soft/rsync/rsyncd.conf

设置文件夹权限：

> chown \-R rsync:rsync /install

阿里云开启8877端口，本机开始同步

客户端\-测试，读取下文件列表：

> rsync \-\-port=8877 \-\-list\-only rsync@8.142.161.156::install

客户端\-测试:

> rsync \-\-port=8877 \-avz \-\-progress /data/new\_centos\_install/\* rsync@59.110.167.206::install

修改配置文件，客户端不允许查看列表：

> write only = yes

开始使用自定义用户验证：

修改配置文件:

```
strict modes = no
auth users = xiaoz
secrets file=/soft/rsync/ps.db
```

创建自定义用户密码文件：

```
touch /soft/rsync/ps.db
install:123456
chmod 600 /soft/rsync/ps.db
```

客户端设置密码：

```
touch /etc/rsync.ps.db
1234 > /etc/rsync.ps.db
chmod 600 /etc/rsync.ps.db
```

> cd /install/zip

rsync \-\-port=8877 \-avz \-\-progress /Users/wangdongyan/Desktop/1.jpg xiaoz@8.142.161.156::install \-\-password\-file=/etc/rsync.ps.db

```
#从远端拉取数据到本地
rsync -auzvP --progress  rsync@118.244.192.100::install /install/src     
rsync -auzvP --progress  --delete rsync@118.244.192.100::tools /tools     
#本地数据同步到远端

--delete:保持同步数据一样，如果多的文件直接DEL
--password-file=rsync.password：LINUX用户登陆 
 
nohup `rsync -auzvP --progress rsync@114.112.64.130::www /www`  > rsync.log &

service xinetd restart
```

#### 分分分分分分隔

被同步的机器：

groupadd rsync;

useradd \-g rsync rsync ;

passwd rsync;test123

su \- rsync;测试一下

同步机器：

mkdir /etc/rsync

touch /etc/rsync/rsyncd.conf

vim rsyncd.conf

```

port = 8877
log file = /data/logs/rsync/rsyncd.log
pid file = /var/rsyncd.pid

#是否检查口令文件的权限
strict modes =yes
port = 873
log file = /etc/rsync/rsyncd.log
pid file = /etc/rsync/rsyncd.pid

#模块名
[install]
max connections = 5
uid = rsync
gid = rsync
path = /root/test_rsync
hosts allow = *
hosts deny = 10.5.3.77
secrets file = /etc/rsync.ps
#####################################################
touch /etc/rsync/rsync.ps
rsync:test123

rsync --daemon --config=/etc/rsyncd.conf
ps aux|grep rsync

从服务端同步数据：backup为服务端的模块名
rsync -auzvP --progress rsync@192.168.0.1::tool /tool
rsync -auzvP --progress rsync@192.168.0.1::www /www

```
