//不是显示当前状态，而是过去多少秒INNODB的状态

show engine innodb status \G

内存：

缓冲池(buffer pool)、(重做)日志缓冲池(redo log buffer)、额外的内存池(additional memory pool)，由innodb_buffer_pool_size 和innodb_log_buffer_size innodb_additional_mem_pool_size配置

show variables like 'innodb_buffer_pool_size'\G

show variables like 'innodb_log_buffer_size'\G

show variables like 'innodb_buffer_pool_size'\G

show variables like 'innodb_additional_mem_pool_size'\G

缓存池占最大部分：存数据，将文件以页(16K)读取到缓冲池，以LRU算法缓存数据。如果修改数据，先修改缓存池中的数据，再FLUSH到硬盘文件

show engine innodb status \G

database pages:一共缓存了多少数据帧

modifyed db pages:脏页

缓冲池缓存的数据页：索引页、数据页、UNDO页、插入缓冲、自适应HASH索引、INNODB存储的锁信息、数据字典等

日志缓冲：主要是重做日志，FLUSH，不建设设置太大，因为大的话，FLUSH的时间就长

额外缓冲：当 主缓冲池内存不够便会申请这个缓冲区

//---------------------------------------------------------------------------------

线程

默认，后台7个线程

4个IO，一个MASTER，一个LOCK监控，一个错误监控

IO thread：由配置文件 innodb_file_io_threads 参数控制，如下

insert buffer thread

log thread

read thread

write thread

LINUX下,IO 4个数量不能调整，WIN可以

版本高了之后，read thread 和 wtire thread 增加到8个，并不用innodb_file_io_threads，而用innodb_read_io_threads,innodb_write_io_threads

show variables like 'innodb_version'\G

show variables like 'innodb_%io_threads'\G

master thread

内部有几个循环：主循环、后台循环、刷新循环、暂停循环（suspend loop）

master会根据不同的情况在几种循环切换

void master_thread(){

loop:

for (i=0;i<10;i++){

do thing once per second

sleep 1 second if necessary

}

do things once per then seconds

goto loop;

}

以上代码中，一个10秒做10次，结尾还有一个10秒做一次，当然这是好的情况，如果负载大了，就会有延迟

每秒一次操作：

日志缓冲刷新到硬盘，即使这个事务还没有提交（总是）

合并插入缓冲（可能，如果一秒内IO次数小于5次，INNODB会认为压力小，进行合并插入缓冲）

至多刷新100个INNODB的缓冲池中的脏页到磁盘（可能，判断buf_get_modified_ratio_pct(脏页比例)是否超过,配件文件的innodb_max_dirty_pages_pct(默认为90%))

如果当前没有用户活动，切换到后台循环（可能）

结合以上，如下伪代码

void master_thread(){

loop:

for (i=0;i<10;i++){

thread_sleep(1);

if (last_one_second_ios <5)

do merage at most 5 insert buffer;

if(buf_get_modified_ratio_pct > innodb_max_dirty_pages_pct)

do buffer pool flush 100 dirty pages

if(no_user_activity)

goto backgroud loop

}

do things once per then seconds

goto loop;

}

接下来看看每10秒做什么：

刷新100脏页到磁盘（可能）

合并至少5个插入缓冲（总是）

将日志缓冲刷新到磁盘（总是）

删除无用的UNDO页（总是）

刷新100脏页个或者10个脏页到磁盘（总是）

产生一个检查点（总是）

以上过程，INNODB先判断过去10秒内IO操作小于200次，刷新100脏页

接着，合并插入缓冲，日志FLUSH

然后，执行full purge,删除无用的UNDO页，一次最多20页

然后，刷新脏页

最后，会产生一个CHECKPOINT点，用于下次10秒操作的点

伪代码：

void master_thread(){

loop:

for (i=0;i<10;i++){

thread_sleep(1);

if (last_one_second_ios <5)

do merage at most 5 insert buffer;

if(buf_get_modified_ratio_pct > innodb_max_dirty_pages_pct)

do buffer pool flush 100 dirty pages

if(no_user_activity)

goto backgroud loop

}

if (last_one_second_ios <200)

do buffer pool flush 100 dirty pages

do merage at most 5 insert buffer;

do log buffer flush disk;

do full purge

if( buf_get_modified_ratio_pct > 70%)

do buffer pool flush 100 dirty pages

else

do buffer pool flush 10 dirty pages

do fuzzy checkpoint

backroup loop:

do something;

goto loop;

}

若无活动用户会跳到backgroup loop

删除无用的UNDO页（总是）

合并20个插入缓冲（总是）

跳到主循环（总是）

不断刷新100个页，直到符合条件（可能，跳到FLUSH LOOP 中）

如果flush loop 也无事可做，就跳到 suspend loop 交master 线程挂起

最后完整的MASTER伪代码：

void master_thread(){

loop:

for (i=0;i<10;i++){

thread_sleep(1);

if (last_one_second_ios <5)

do merage at most 5 insert buffer;

if(buf_get_modified_ratio_pct > innodb_max_dirty_pages_pct)

do buffer pool flush 100 dirty pages

if(no_user_activity)

goto backgroud loop

}

if (last_one_second_ios <200)

do buffer pool flush 100 dirty pages

do merage at most 5 insert buffer;

do log buffer flush disk;

do full purge

if( buf_get_modified_ratio_pct > 70%)

do buffer pool flush 100 dirty pages

else

do buffer pool flush 10 dirty pages

do fuzzy checkpoint

goto loop;

backroup loop:

do full purge

do marge 20 insert buffer

if not idle:

go to loop

else :

goto flush loop

flush loop:

do buffer pool flush 100 dirty pages

if(buf_get_modified_ratio_pct > innodb_max_dirty_pages_pct)

goto flush loop;

goto suspend loop;

suspend loop:

suspend _thred();

waiting event

goto loop;

}

show engine innodb status \G

srv_master_thread loops(每秒，每十秒的情况)

srv_master_thread log flush and write

问题：每秒flush 到磁盘是有数的，而在现在SSD普及的情况下，刷新的速度是可以再提高的

之后，INNODB 做了升级，加了个参数，innodb_io_capacity,规则如下：

合并插入缓冲时，数量为innodb_io_capacity的5%

刷新脏页数为innodb_io_capacity

如果用SSD 的话，可以调高此值

又增加了一个参数innodb_adaptive_flushing(自适应的刷新),影响每一秒刷新脏页数量，而不是十秒的，切记

原规则：脏页缓冲池的比例大于 参数设置百分比，flush 脏页到磁盘，小于则不flush

现在：新函数 buf_flush_get_desired_flush_rate ，判断重做日志的速度来判断最合适的刷新脏页数量，而如果上面不flush的时候，调用buf_flush_get_desired_flush_rate也有可能flush

innodb_fast_shutdown:

0:完全关闭， full purge merge isnert buffer等

1:默认值，不做 full purge merge isnert buffer等，但是脏页要写到磁盘上

2:什么都不做，脏页写到日志文件，待重启再修复磁盘文件

innodb_force_recovery

第三章

参数分为：动态（dynimic:运行期间不允许改），静态:运行期间可以更改

动态参数：又分：global session

show variables like 'log_error';

错误日志，慢查询日志，查询日志，二进制日志

slow log:

show variables like '%login_query_time',默认是10秒,5.1以上支持小数,可存表中，mysql.slow_log,CVS引擎，可改成MYISAM

show variables like 'log_slow_queries',默认是关闭

show variables like 'log_queries_not_using_indexes'，未使用索引的也记录日志中

show variables like 'log_output',默认FILE，可改成TABLE

mysqldumpslow 分析工具

查询日志：select| show 等查询的QUERY，5.1之后也支持表mysql.general_log

二进制日志，不包含select 、show 这样的操作，因为没有修改，主要作用： 恢复与复制

*.index文件，为索引文件，每次新生成一个文件便会记录其中

max_binlog_size:单个二进制文件大小，如果超过则产生新文件，后缀名+1，并记录，默认为1GB

binlog_cache_size：事务未提交的QUERY，会被记录到一个缓存中，提交后 flush,show global status:binlog_cache_use ,binlog_Cache_disk_use,如果值太小会写入临时文件

sync_binlog：默认0，写入缓冲中再FLUSH磁盘，1：写缓冲同步写到磁盘上，如果为1，这样从库复制就非常会，不用等缓冲FLUSH，但对IO影响 ~不过，为1，当事务回滚，二进制文件不能回滚，innodb_support_xa

binlog-do-db:哪些DB需要写入文件

binlog-ingore-db:忽略哪些DB写入文件

log-slave-update

binlog_format：

statement：普通的SQL QUERY，缺点：rand uniq 随机函数主从数据不一致 ,UPDATE可能主丛不一致

row:除了SQL QUERY，还添加了表修改信息，解决了上面问题，但是就是文件太大，尤其从库复制传输，隔离：read commited

mixed：是结合 了以上两种，正常是statement，当有特殊情况转换成ROW(uuid,user,current_user,found_rows,row_count),使用了temporary table

sock文件

PID文件

myisam:frm myi myd

INNODB:

表空间

默认文件名：ibdata1

show variables like ='innodb_data_file_path'

默认是一个文件，是可以设置成两个文件，提高性能,autoextend 自增

innodb_file_per_table：此参数默认为OFF,如果打开，每个表都有一个表空间文件 .ibd,但此文件只保存 数据 插入缓冲等信息，其余信息还是存在公共表文件中,如：UNDO，系统事务，二次缓冲写等

重做日志文件(redo log)：ib_logfile0 ib_logfile1

同样，默认是俩个文件，称为一组，可以设置多组，且放在不同的磁盘上面,他是循环的做日志，0满了，往1写，1满了再往0写

innodb_log_file_size

innodb_log_file_in_group:一组中有几个文件

innodb_mirrored_log_groups:设置几个组

innodb_log_group_home_dir:目录位置

重做日志

flush_log_at_trx_commit

事务提交时：

0：不写入日志文件，而是等待主线程每秒刷新

1：写入磁盘

2：异到写入硬盘

//第4章

主键，聚集索引，如果建表时不设置：

1，找寻唯一且非NULL 的字段

2，自动创建一个6字节大小的指针

表空间由：段，区，页组成

undo,执行回滚后，文件并没有变小，只是标志某行记录为不可用

段：又分为索引段数据段等

个人觉得INNODB的表空间有点像CPU的 各种段存储的方式

**89