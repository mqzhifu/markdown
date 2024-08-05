```

[mysqld]

# Mysql服务的唯一编号 每个mysql服务Id需唯一

server-id = 1

  

# 允许访问的IP网段

bind-address = 0.0.0.0

  

# 服务端口号 默认3306

port = 3306

  

# 用户

user = mysql

  

# mysql安装根目录

basedir = /soft/mysql

  

datadir = /data/mysql/master_3306

  

pid-file = /data/mysql/master_3306/mysql.pid

  

log_error = /data/mysql/master_3306/error.log

  

socket = /tmp/mysql.sock

  

#必须得写，不然密码验证，PHP连接不上

default_authentication_plugin = mysql_native_password

  

#sql-mode="NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION"

#sql_mode = STRICT_TRANS_TABLES, NO_ZERO_IN_DATE, NO_ZERO_DATE, ERROR_FOR_DIVISION_BY_ZERO, NO_AUTO_CREATE_USER, NO_ENGINE_SUBSTITUTION

sql_mode='STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION'

  

# 只能用IP地址检查客户端的登录，不用主机名

#skip_name_resolve = 1

  

# 事务隔离级别，默认为可重复读，mysql默认可重复读级别（此级别下可能参数很多间隙锁，影响性能）

transaction_isolation = READ-COMMITTED

  

# 数据库默认字符集,主流字符集支持一些特殊表情符号（特殊表情符占用4个字节）

#character-set-server = utf8

character-set-server=utf8mb4

  

# 数据库字符集对应一些排序等规则，注意要和character-set-server对应

#collation-server = utf8_general_ci

collation_server=utf8mb4_unicode_ci

  

# 设置client连接mysql时的字符集,防止乱码

init_connect='SET NAMES utf8mb4'

  

# 是否对sql语句大小写敏感，1表示不敏感

#lower_case_table_names = 1

  

# 最大连接数

max_connections = 400

back_log = 600

  

# 最大错误连接数

max_connect_errors = 1000

  

# TIMESTAMP如果没有显示声明NOT NULL，允许NULL值

explicit_defaults_for_timestamp = true

  

# SQL数据包发送的大小，如果有BLOB对象建议修改成1G

max_allowed_packet = 128M

  

  

# MySQL连接闲置超过一定时间后(单位：秒)将会被强行关闭

# MySQL默认的wait_timeout 值为8个小时, interactive_timeout参数需要同时配置才能生效

interactive_timeout = 1800

wait_timeout = 1800

  

# 内部内存临时表的最大值 ，设置成128M。

# 比如大数据量的group by ,order by时可能用到临时表，

# 超过了这个值将写入磁盘，系统IO压力增大

tmp_table_size = 134217728

max_heap_table_size = 134217728

  

# 慢查询sql日志设置

# 检索的行数必须达到此值才可被记为慢查询

min_examined_row_limit = 100

slow_query_log = 5

slow_query_log_file = /data/mysql/master_3306/slow.log

  

# 检查未使用到索引的sql

log_queries_not_using_indexes = 1

  

# 针对log_queries_not_using_indexes开启后，记录慢sql的频次、每分钟记录的条数

log_throttle_queries_not_using_indexes = 5

  

# 作为从库时生效,从库复制中如何有慢sql也将被记录

log_slow_slave_statements = 1

  

#binlog

  

#mysql默认采用statement，建议使用mixed

binlog_format = MIXED

log-bin = /data/mysql/master_3306/mysql-bin.log

#binlog过期清理时间 - 7天

binlog_expire_logs_seconds = 604800

#binlog每个日志文件大小

max_binlog_size = 100m

#binlog缓存大小

binlog_cache_size = 4m

#最大binlog缓存大小

max_binlog_cache_size = 100m

  

  

```