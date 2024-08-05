QPS

MySQL Server 每秒执行的 Query总量

QPS = Questions(or Queries) / Seconds

最大连接

MySQL: ERROR 1040: Too many connections(error)

mysql> show variables like ‘max_connections‘;

索引读请求未命中

key_buffer_size是对MyISAM表性能影响最大的一个参数

mysql> show variables like ‘key_buffer_size‘;

mysql> show global status like ‘key_read%‘;

+------------------------+-------------+

| Variable_name　　　　　　　　　　 | Value　　　　　　　 |

+------------------------+-------------+

| Key_read_requests　　　　　　 | 27813678764 |

| Key_reads　　　　　　　　　　　　　　 | 6798830　　　　　 |

+------------------------+-------------+

一共有27813678764个索引读取请求，有6798830个请求在内存中没有找到直接从硬盘读取索引，计算索引未命中缓存的概率：

key_cache_miss_rate ＝ Key_reads / Key_read_requests * 100%

Innodb Buffer 命中率

Innodb Buffer-》innodb_buffer_pool,缓存 Innodb 类型表的数据和索引的内存空间。类似 Key buffer,我们同样可以根据 MySQL Server 提供的相应状态值计算出其命中率:

innodb_buffer_read_hits=(1-Innodb_buffer_pool_reads/Innodb_buffer_pool_read_requests) * 100%

获取所需状态变量值:

sky@localhost : (none) 08:25:14> SHOW /*!50000 GLOBAL*/ STATUS

-> LIKE 'Innodb_buffer_pool_read%';

+-----------------------------------+-------+

| Variable_name | Value |

+-----------------------------------+-------+

... ...

| Innodb_buffer_pool_read_requests | 5367 |

| Innodb_buffer_pool_reads | 507 |

+-----------------------------------+-------+

临时表

mysql> show global status like ‘created_tmp%‘;

+-------------------------+---------+

| Variable_name　　　　　　　　　　　 | Value　　　 |

+-------------------------+---------+

| Created_tmp_disk_tables | 21197　　　 |

| Created_tmp_files　　　　　　　 | 58　　　　　　 |

| Created_tmp_tables　　　　　　 | 1771587 |

+-------------------------+---------+

每次创建临时表，Created_tmp_tables增加，如果是在磁盘上创建临时表，Created_tmp_disk_tables也增加,Created_tmp_files表示MySQL服务创建的临时文件文件数，比较理想的配置是：

　　Created_tmp_disk_tables / Created_tmp_tables * 100% <= 25%比如上面的服务器Created_tmp_disk_tables / Created_tmp_tables * 100% ＝ 1.20%，应该相当好了。我们再看一下MySQL服务器对临时表的配置：

mysql> show variables where Variable_name in (‘tmp_table_size‘, ‘max_heap_table_size‘);

+---------------------+-----------+

| Variable_name　　　　　　　 | Value　　　　　 |

+---------------------+-----------+

| max_heap_table_size | 268435456 |

| tmp_table_size　　　　　　 | 536870912 |

+---------------------+-----------+

mysql> show global status like ‘open%tables%‘;

打开表缓存

Open_tables表示打开表的数量，Opened_tables表示打开过的表数量，如果Opened_tables数量过大，说明配置中 table_cache(5.1.3之后这个值叫做table_open_cache)值可能太小，我们查询一下服务器table_cache值：

mysql> show variables like ‘table_cache‘;

//进程使用情况

mysql> show global status like ‘Thread%‘;

+-------------------+-------+

| Variable_name　　　　　 | Value |

+-------------------+-------+

| Threads_cached　　　　 | 46　　　　 |

| Threads_connected | 2　　　　　 |

| Threads_created　　　 | 570　　　 |

| Threads_running　　　 | 1　　　　　 |

+-------------------+-------+

如果发现Threads_created值过大的话

mysql> show variables like ‘thread_cache_size‘;

Thread Cache 命中率

Thread Cache 命中率:Thread Cache 命中率能够直接反应出我们的系统参数，thread_cache_size 设置的是否合理。一个合理的 thread_cache_size 参数能够节约大量创建新连接时所需要消耗的资源。Thread Cache 命中率计算方式如下:

Thread_cache_hits = (1 - Threads_created / Connections) * 100%

获取所需状态变量值:

sky@localhost : (none) 08:57:16> SHOW /*!50000 GLOBAL*/ STATUS LIKE 'Thread%';

sky@localhost : (none) 09:01:33> SHOW /*!50000 GLOBAL*/ STATUS LIKE 'Connections';

正常来说,Thread Cache 命中率要在 90% 以上才算比较合理。

锁定状态:

SHOW STATUS LIKE '%lock%';

+-------------------------------+-------+

| Variable_name | Value |

+-------------------------------+-------+

... ...

| Innodb_row_lock_current_waits | 0 |

| Innodb_row_lock_time | 0 |

| Innodb_row_lock_time_avg | 0 |

| Innodb_row_lock_time_max | 0 |

| Innodb_row_lock_waits | 0 |

... ...

| Table_locks_immediate | 44|

| Table_locks_waited | 0 |

+-------------------------------+-------+

行锁、表锁、造成线程等待、锁定时间

查询缓存(query cache)

mysql> show global status like ‘qcache%‘;

Qcache_free_blocks：缓存中相邻内存块的个数。数目大说明可能有碎片。FLUSH QUERY CACHE会对缓存中的碎片进行整理，从而得到一个空闲块。

Qcache_free_memory：缓存中的空闲内存。

Qcache_hits：每次查询在缓存中命中时就增大

Qcache_inserts：每次插入一个查询时就增大。命中次数除以插入次数就是不中比率。

Qcache_lowmem_prunes：缓存出现内存不足并且必须要进行清理以便为更多查询提供空间的次数。这个数字最好长时间来看；如果这个数字在不断增长，就表示可能碎片非常严重，或者内存很少。（上面的 free_blocks和free_memory可以告诉您属于哪种情况）

Qcache_not_cached：不适合进行缓存的查询的数量，通常是由于这些查询不是 SELECT 语句或者用了now()之类的函数。

Qcache_queries_in_cache：当前缓存的查询（和响应）的数量。

Qcache_total_blocks：缓存中块的数量。

我们再查询一下服务器关于query_cache的配置：

mysql> show variables like ‘query_cache%‘;

各字段的解释：

query_cache_limit：超过此大小的查询将不缓存

query_cache_min_res_unit：缓存块的最小大小

query_cache_size：查询缓存大小

query_cache_type：缓存类型，决定缓存什么样的查询，示例中表示不缓存 select sql_no_cache 查询

query_cache_wlock_invalidate：当有其他客户端正在对MyISAM表进行写操作时，如果查询在query cache中，是否返回cache结果还是等写操作完成再读表获取结果。

query_cache_min_res_unit的配置是一柄”双刃剑”，默认是4KB，设置值大对大数据查询有好处，但如果你的查询都是小数据查询，就容易造成内存碎片和浪费。

查询缓存碎片率 = Qcache_free_blocks / Qcache_total_blocks * 100%

如果查询缓存碎片率超过20%，可以用FLUSH QUERY CACHE整理缓存碎片，或者试试减小query_cache_min_res_unit，如果你的查询都是小数据量的话。

查询缓存利用率 = (query_cache_size - Qcache_free_memory) / query_cache_size * 100%

查询缓存利用率在25%以下的话说明query_cache_size设置的过大，可适当减小；查询缓存利用率在80％以上而且Qcache_lowmem_prunes > 50的话说明query_cache_size可能有点小，要不就是碎片太多。

查询缓存命中率 = (Qcache_hits - Qcache_inserts) / Qcache_hits * 100%

示例服务器 查询缓存碎片率 ＝ 20.46％，查询缓存利用率 ＝ 62.26％，查询缓存命中率 ＝ 1.94％，命中率很差，可能写操作比较频繁吧，而且可能有些碎片。

//排序

show global status like 'sort%';

Sort_merge_passes 包括两步。MySQL 首先会尝试在内存中做排序，使用的内存大小由系统变量 Sort_buffer_size 决定，如果它的大小不够把所有的记录都读到内存中，MySQL 就会把每次在内存中排序的结果存到临时文件中，等 MySQL 找到所有记录之后，再把临时文件中的记录做一次排序。这再次排序就会增加 Sort_merge_passes。实际上，MySQL 会用另一个临时文件来存再次排序的结果，所以通常会看到 Sort_merge_passes 增加的数值是建临时文件数的两倍。因为用到了临时文件，所以速度可能会比较慢，增加 Sort_buffer_size 会减少 Sort_merge_passes 和 创建临时文件的次数。但盲目的增加 Sort_buffer_size 并不一定能提高速度

http://qroom.blogspot.com/2007/09/mysql-select-sort.html

http://www.mysqlperformanceblog.com/2007/07/24/what-exactly-is-read_rnd_buffer_size/

文件打开数(open_files)

show global status like ‘open_files‘;

mysql> show variables like ‘open_files_limit‘;

Open_files / open_files_limit * 100% <= 75％

表锁

show global status like ‘table_locks%‘;

　　Table_locks_immediate表示立即释放表锁数，Table_locks_waited表示需要等待的表锁数，如果 Table_locks_immediate / Table_locks_waited > 5000，最好采用InnoDB引擎，因为InnoDB是行锁而MyISAM是表锁，对于高并发写入的应用InnoDB效果会好些。示例中的服务器 Table_locks_immediate / Table_locks_waited ＝ 235，MyISAM就足够了。

扫表

show global status like ‘handler_read%‘;

http://hi.baidu.com/thinkinginlamp/blog/item/31690cd7c4bc5cdaa144df9c.html

计算表扫描率：

表扫描率 ＝ Handler_read_rnd_next / Com_select

如果表扫描率超过4000，说明进行了太多表扫描，很有可能索引没有建好，增加read_buffer_size值会有一些好处，但最好不要超过8MB。

（原文：http://www.001pp.com/chengxuyouhua/mysql%20xingnengyouhua2183.html）