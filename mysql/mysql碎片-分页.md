MYISAM，删除一条数据后，实际并不是物理删除，而是给该行打上删除标记，当有新的插入里覆盖此行

这些删除标记的记录，很浪费空间

information_schema tables:data_free 字段查看

ANALYZE TABLE

SHOW INDEX FROM tablename 查看索引散列程度

myisamchk注意使用 myisamchk 时要停止 MySQL

mysqlcheck

REPAIR

OPTIMIZE TABLE tablename ,会锁表

可以先修复从库，主库继续各种业务，之后切换，再修，完成了

大数据量分页：

两条不好SQL：

SELECT * FROM table LIMIT 1000, 10;

SELECT * FROM table LIMIT 10000000, 10;

limit 和数据表的大小有关的

//优化1

SELECT * FROM table ORDER BY id LIMIT 1000000, 10;

让其ID走索引

//优化2-1

SELECT * FROM table WHERE id >= (SELECT id FROM table LIMIT 1000000, 1) LIMIT 10;

//优化2-2

SELECT * FROM table WHERE id BETWEEN 1000000 AND 1000010;

select id from collect where vtype=1 order by id limit 90000,10; 很慢，用了8-9秒！

//优化3

添加索引表，数据量大了还会慢

//复合索引

1. 经常变化的字段用char

2. 知道固定长度的用char

3. 尽量用varchar

4. 超过255字节的只能用varchar或者text

5. 能用varchar的地方不用text

如果原表中都是char，只要把其中一个字段改为varchar或者text，则其它所有char字段自动改为varchar