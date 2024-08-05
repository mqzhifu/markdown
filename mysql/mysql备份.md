主键索引(聚集)、二级索引(非聚集)

DML操作是指对数据库中表记录的操作，主要包括表记录的插入（insert）、更新（update）、删除（delete）、查询（select）

WAL(write-ahead logging):日志先行

LSN(log sequence number):日志序列号

http://www.percona.com/

xtrabackup(innobackupex)

mysqldump:几个重要参数

--all-databases , -A,导出所有数据库

--lock-all-tables：它请求发起一个全局的读锁，会阻止对所有库的表的写入操作，以此来确保数据的一致性

--lock-tables：它锁定当前导出的数据库库数据表，而不是一下子锁定全部库下的表。本选项只适用于 MyISAM 表，如果是 Innodb 表可以用 --single-transaction 选项。

--SQL_NO_CACHE 来确保不会读取缓存里的数据

--single-transaction

InnoDB 表在备份时，通常启用选项 --single-transaction 来保证备份的一致性，实际上它的工作原理是设定本次会话的隔离级别为：

REPEATABLE READ，以确保本次会话(dump)时，不会看到其他会话已经提交了的数据。

--no-data，-d:不导出数据

--complete-insert, -c：使用完整的insert语句(包含列名称)。这么做能提高插入效率，但是可能会受到max_allowed_packet参数的影响而导致插入失败。

--add-drop-database：每个数据库创建之前添加drop数据库语句。

--add-drop-table：每个数据表创建之前添加drop数据表语句。(默认为打开状态，使用--skip-add-drop-table取消选项)

--create-options, -a：在CREATE TABLE语句中包括所有MySQL特性选项。(默认为打开状态)

--debug：输出debug信息，用于调试。默认值为：d:t:o,/tmp/mysqldump.trace

--default-character-set=utf8

--disable-keys：告诉 MySQLdump 在 INSERT 语句的开头和结尾增加 /*!40000 ALTER TABLE table DISABLE KEYS */; 和 /*!40000 ALTER TABLE table ENABLE KEYS */; 语句，这能大大提高插入语句的速度，因为它是在插入完所有数据后才重建索引的。该选项只适合 MyISAM 表。

--quick，-q：该选项在导出大表时很有用，它强制 MySQLdump 从服务器查询取得记录直接输出而不是取得所有记录后将它们缓存到内存中。

--extended-insert, -e：使用具有多个VALUES列的INSERT语法。这样使导出文件更小，并加速导入时的速度。默认为打开状态，使用--skip-extended-insert取消选项。

--log-error：附加警告和错误信息到给定文件

--set-charset：添加'SET NAMES default_character_set'到输出文件。默认为打开状态，使用--skip-set-charset关闭选项。

--master-data：该选项将binlog的位置和文件名追加到输出文件中。如果为1，将会输出CHANGE MASTER 命令；如果为2，输出的CHANGE MASTER命令前添加注释信息。该选项将打开--lock-all-tables 选项，除非--single-transaction也被指定（在这种情况下，全局读锁在开始导出时获得很短的时间；其他内容参考下面的--single-transaction选项）。该选项自动关闭--lock-tables选项。

--single-transaction：该选项在导出数据之前提交一个BEGIN SQL语句，BEGIN 不会阻塞任何应用程序且能保证导出时数据库的一致性状态。它只适用于多版本存储引擎，仅InnoDB。本选项和--lock-tables 选项是互斥的，因为LOCK TABLES 会使任何挂起的事务隐含提交。要想导出大表的话，应结合使用--quick 选项。

--dump-date：将导出时间添加到输出文件中。默认为打开状态，使用--skip-dump-date关闭选项。

--delete-master-logs：master备份后删除日志. 这个参数将自动激活--master-data。清除以前的日志，以释放空间。但是如果服务器配置为镜像的复制主服务器，删掉MySQL二进制日志很危险，因为从服务器可能还没有完全处理该二进制日志的内容。在这种情况下，使用 PURGE MASTER LOGS更为安全。

--flush-logs：在mysqldump的时候指定--flush-logs选项，它的含义是在备份结束后会结束当前的binlog，生成一个新的binlog，而这个新的binlog，就是你所谓的增量。

--opt：这只是一个快捷选项，等同于同时添加

--add-drop-tables

--add-locking

--create-option

--disable-keys

--extended-insert

--lock-tables

--quick

--set-charset

选项。本选项能让 MySQLdump 很快的导出数据，并且导出的数据能很快导回。该选项默认开启，但可以用 --skip-opt 禁用。注意，如果运行 MySQLdump 没有指定 --quick 或 --opt 选项，则会将整个结果集放在内存中。如果导出大数据库的话可能会出现问题。

DDL is NOT transactional ！DDL ,如alert table,并不是事务，所以在备份时--single-transaction 参数是无效的，必须要--lock-all-tables