bin-log :有3种模式

statement：记录SQL语句，及该条语句的上下文信息

row：记录实际的数据，比如update 整张表，那就记录整张表的所有 修改数据~而不是记录update 语句

mixed：结合了上面两种，偶尔是statement 偶尔是row

statement：不是所有 UPDATE 被复制，像 UDF用户自定义函数，还有一些特定函数

几个相关的参数：

binlog_format = MIXED //binlog日志格式

log_bin =目录/mysql-bin.log //binlog日志名

expire_logs_days = 7 //binlog过期清理时间

max_binlog_size 100m //binlog每个日志文件大小

1修改主库配置文件

log-bin = /home/mysql/log/mysql-bin.log

server-id=1

2 分别在主从各建立一个同步账户

use mysql;

truncate table user;

grant ALL on *.* to 'root'@'localhost' identified by 'root' WITH GRANT OPTION;

grant ALL on *.* to 'root'@'%' identified by 'root' WITH GRANT OPTION;

grant SELECT, REPLICATION CLIENT, REPLICATION SLAVE on *.* to 'slave'@'%' identified by 'slave';

3找到当前点

mysql> show master status\G;

File: mysql-bin.000003

Position: 243

4停止主库进程

5修改从库配置文件

server-id=2

log_bin = /var/log/mysql/mysql-bin.log

master-host =192.168.1.100

master-user=test

master-pass=123456

master-port =3306

master-connect-retry=60

replicate-do-db =test #要同步的数据库

6启动同步

mysql> start slave;

innodb_flush_log_at_trx_commit=1

OAD DATA FROM MASTER

LOAD TABLE FROM MASTER

SELECT @@sql_mode

http://www.cnblogs.com/ainiaa/archive/2010/12/31/1923002.html