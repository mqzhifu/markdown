# 鲸鱼DB

**设计规范与目的**

 MySQL数据库与Oracle、sqlserver、PostgreSQL等数据库相比，有其内核上的优势与劣势。我们在使用MySQL数据库的时候需要遵循一定规范，扬长避短。本规范是在帮助或指导RD、QA、OP等技术人员做出适合线上业务的数据库设计。在数据库变更和处理流程、数据库表设计、SQL编写等方面予以规范，从而为公司业务系统稳定、健康地运行提供保障。

**设计规范**

- **数据库设计**

 以下所有规范会按照【高危】、【强制】、【建议】三个级别进行标注，遵守优先级从高到低。对于【高危】和【强制】两个级别的设计，DBA会强制打回要求修改。

- **库名**

1. 【强制】库的名称必须控制在32个字符以内，相关模块的表名与表名之间尽量体现join的关系，如user表和user\_login表。
2. 【强制】库的名称格式：业务系统名称\_子系统名，同一模块使用的表名尽量使用统一前缀。
3. 【强制】创建数据库时必须显式指定字符集，并且字符集只能是utf8或者utf8mb4

创建数据库SQL举例：

            Create database db1 default character set utf8;

- **表结构**

1. 【强制】表和列的名称必须控制在32个字符以内，表名只能使用字母、数字和下划线，一律小写。
2. 【强制】表中列的个数必须控制40个以内，否则表的行长的不可控，查询对磁盘IO 和网卡IO开销太大，影响SQL查询效率，严重情况用户查询响应慢。
3. 【强制】表名要求模块名强相关，如师资系统采用”sz”作为前缀，渠道系统采用”qd”作为前缀等。
4. 【强制】创建表时必须显式指定字符集为utf8或utf8mb4。
5. 【强制】创建表时必须显式指定表存储引擎类型，如无特殊需求，一律为InnoDB。当需要使用除InnoDB/MyISAM/Memory以外的存储引擎时，必须通过DBA审核才能在生产环境中使用。因为Innodb表支持事务、  行锁、宕机恢复、MVCC等关系型数据库重要特性，其他存储引擎没有这些特性，因此首推Innodb。
6. 【强制】建表和表结构必须有comment
7. 【建议】建表时关于主键：

         \(1\)强制要求主键为id，类型为int或bigint，且为auto\_increment。

         \(2\)标识表里每一行主体的字段不要设为主键，建议设为其他字段如user\_id，order\_id等，并建立unique key索引。因为如果设为主键且主键值为随机插入，则会导致innodb内部page分裂和大量随机I/O，性能下降。

1. 【强制】核心表（如用户表，金钱相关的表）必须有行数据的创建时间字段create\_time和最后更新时间字段update\_time，便于查问题。
2. 【强制】表中所有字段必须都是NOT NULL属性，业务可以根据需要定义DEFAULT值。 因为使用NULL值会存在每一行都会占用额外存储空间、索引效率下降、数据迁移容易出错、聚合函数计算结果偏差等问题。
3. 【建议】建议对表里的blob、text等大字段，垂直拆分到其他表里，仅在需要读这些对象的时候才去select。用不到大对象，千万不要去创建，表的行长度会变长，查询性能直线下降。
4. 【建议】反范式设计：把经常需要join查询的字段，在其他表里冗余一份。如user\_name属性在user\_account，user\_login\_log等表里冗余一份，减少join查询。
5. 【强制】中间表用于保留中间结果集，名称必须以“tmp\_”开头。
6.                备份表用于备份或抓取源表快照，名称必须以“bak\_”开头。

               中间表和备份表定期清理。

1. 【强制】对于超过100W行的大表进行alter table，必须经过DBA审核，并在业务低峰期执行。因为alter table会产生表锁，期间阻塞对于该表的所有写入，对于业务可能会产生极大影响。

- **列数据类型优化**

1. 【建议】表中的自增列无符号（auto\_increment属性），推荐使用bigint类型。因为int存储范围为\-2147483648~2147483647（大约21亿左右），溢出后会导致报错。
2. 【建议】业务中选择性很少的状态status、类型type等字段推荐使用tinytint或者smallint类型节省存储空间。
3. 【建议】业务中IP地址字段推荐使用int类型，不推荐用char\(15\)，因为int只占4字节，可以用如下函数相互转换，而char\(15\)占用至少15字节。一旦表数据行数到了1亿，那么要多用1.1G存储空间！
4. 【建议】不推荐使用enum，set 因为它们浪费空间，且枚举值写死了，变更不方便。推荐使用tinyint或smallint
5. 【建议】不推荐使用blob，text等类型，它们都比较浪费硬盘和内存空间。在加载表数据时，会读取大字段到内存里从而浪费内存空间，影响系统性能。建议和PM、RD沟通，是否真的需要这么大字段，

  Innodb中当一行记录超过8098字节时，会将该记录中选取最长的一个字段将其768字节放在原始page里，该字段余下内容放在overflow\-page里。不幸的是在compact行格式下，原始page和overflow\-page都会 加载。

1. 【强制】禁止json数据类型的使用，如果有类似的多字符串统计查询或多字符串的批量查询，建议到ES中去做检索。
2. 【建议】存储金钱的字段，建议用int，程序端乘以100和除以100进行存取，因为int占用4字节，而double占用8字节，空间浪费。
3. 【建议】文本数据尽量用varchar存储，因为varchar是变长存储，比char更省空间。MySQL server层规定一行所有文本最多存65535字节，因此在utf8字符集下最多存21844个字符，超过会自动转换为mediumtext字段。而text在utf8字符集下最多存21844个字符，mediumtext最多存2^24/3个字符，longtext最多存2^32个字符。一般建议用varchar类型，字符数不要超过2700。
4. 【强制】在建表，修改列，新增列时，列属性要有默认值，B\-tree索引 is null不会走索引。做好提前的评估，如果是索引列，建表或添加字段时，不要用default null。最好是not null default '1' COMMENT ='XXX'
5. 【建议】时间类型尽量选取timestamp，因为datetime占用8字节，timestamp仅占用4字节，但是范围为1970\-01\-01 00:00:01到2038\-01\-01 00:00:00。更为高阶的方法，选用int来存储时间，使用SQL函数unix\_timestamp\(\)和from\_unixtime\(\)来进行转换。详细存储大小参数如下图

![d254078fc63a924359eab81a56743621.jpg](image/d254078fc63a924359eab81a56743621.jpg)

 

- **索引设计**

1. 【强制】InnoDB表必须主键为id int/bigint auto\_increment,且主键值禁止被更新。
2. 【建议】唯一键以“uniq\_”开头，普通索引以“idx\_”开头，一律使用小写格式，以表名/字段的名称或缩写作为后缀。
3. 【建议】单个索引中每个索引记录的长度不能超过64KB\(65536字节\)。
4. 【建议】单个表上的索引个数不能超过7个。
5. 【建议】在建立索引时，多考虑建立联合索引，并把区分度最高的字段放在最前面。如列userid的区分度可由select count\(distinct userid\)计算出来。
6. 【强制】在多表join的SQL里，保证被驱动表的连接列上有索引，这样join执行效率最高，切记，驱动表如果是一张大表，执行查询效率很低，禁止使用，即使关联表，不要超过三张表，且表为小表
7. 【建议】建表或加索引时，保证表里互相不存在冗余索引。对于MySQL来说，如果表里已经存在key\(a,b\)，则key\(a\)为冗余索引，需要删除。

- **一个规范的建表语句**

一个较为规范的建表语句为：

CREATE TABLE user \(

\`id\` bigint\(11\) unsigned NOT NULL AUTO\_INCREMENT COMMENT '自增id',

\`user\_id\` bigint\(11\) NOT NULL DEFAULT '0' COMMENT '用户id',

\`username\` varchar\(45\) NOT NULL DEFAULT '' COMMENT '真实姓名',

\`email\` varchar\(30\) NOT NULL DEFAULT '' COMMENT '用户邮箱',

\`nickname\` varchar\(45\) NOT NULL DEFAULT '' COMMENT '昵称',

\`avatar\` int\(11\) NOT NULL DEFAULT '0' COMMENT '头像',

\`birthday\` date NOT NULL DEFAULT '1994\-01\-01' COMMENT '生日',

\`sex\` tinyint\(4\) NOT NULL DEFAULT '0' COMMENT '性别',

\`short\_introduce\` varchar\(150\) NOT NULL DEFAULT '' COMMENT '一句话介绍自己，最多50个汉字',

\`user\_resume\` varchar\(300\) NOT NULL DEFAULT '' COMMENT '用户提交的简历存放地址',

\`user\_register\_ip\` int \(11\) NOT NULL DEFAULT '0' COMMENT '用户注册时的源ip',

\`create\_time\` timestamp NOT NULL DEFAULT CURRENT\_TIMESTAMP COMMENT '创建时间',

\`update\_time\` timestamp NOT NULL DEFAULT CURRENT\_TIMESTAMP COMMENT '用户资料修改的时间',

\`user\_review\_status\` tinyint \(1\) NOT NULL DEFAULT '0' COMMENT '用户资料审核状态，1为通过，2为审核中，3为未通过，4为还未提交审核',

PRIMARY KEY \(\`id\`\),

UNIQUE KEY \`uniq\_user\_id\` \(\`user\_id\`\),

KEY \`idx\_username\`\(\`username\`\),

KEY \`idx\_create\_time\`\(\`create\_time\`,\`user\_review\_status\`\)

\) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='网站用户基本信息';

**SQL编写**

- **DML语句**

1. 【强制】SELECT语句必须指定具体字段名称，禁止写成“\*”，因为select \*会将不该读的数据也从MySQL里读出来，造成网卡压力。且表字段一旦更新，但model层没有来得及更新的话，系统会报错。
2. 【强制】生产环境禁止使用insert into select SQL语句
3. 【强制】insert语句指定具体字段名称，不要写成insert into t1 values\(…\)，道理同上。同时忽略，id自增列，不要手动给主键自增插入数据。
4. 【建议】insert into…values\(XX\),\(XX\),\(XX\).. 这里XX的值不要超过1000个，值过多虽然上线很很快，但会引起主从同步延迟。
5. 【建议】in值列表限制在500以内。例如select… where userid in\(….500个以内…\)，这么做是为了减少底层扫描，减轻数据库压力从而加速查询
6. 【建议】事务里批量更新数据需要控制数量，进行必要的sleep，做到少量多次。
7. 【强制】写入和事务发往主库，只读SQL发往从库。
8. 【强制】除静态表或小表（100行以内），DML语句必须有where条件，且使用索引查找。
9. 【强制】生产环境禁止使用hint，如sql\_no\_cache，force index，ignore key，straight join等，因为hint是用来强制SQL按照某个执行计划来执行，但随着数据量变化我们无法保证自己当初的预判是正确的，因此我们要遵循MySQL优化器！
10. 【强制】where条件里等号左右字段类型必须一致，否则无法利用索引。
11. 【强制】SELECT|UPDATE|DELETE|要有WHERE子句，且WHERE子句的条件必需使用索引查找。
12. 【强制】生产数据库中强烈不推荐大表上发生全表扫描，但对于100行以下的静态表可以全表扫描。
13. 【强制】WHERE 子句中禁止只使用全模糊的LIKE条件进行查找，必须有其他等值或范围查询条件，否则无法利用索引 。
14. 【建议】索引列不要使用函数或表达式，否则无法利用索引。如where length\(name\)=’Admin’或where user\_id\+2=10023
15. 【建议】减少使用or语句，虽然where条件上已建立索引。如where a=1 or b=2，index merge的效果不好，尽量少用
16. 【建议】分页查询，当limit起点较高时，可先用过滤条件进行过滤。

  如select a,b,c from t1 limit 10000,20;

  优化为:

  select a,b,c from t1 where id\>10000 limit 20;

- **多表连接**

1. 【高危】禁止跨db的join语句。因为这样可以减少模块间耦合，为数据库拆分奠定坚实基础。
2. 【强制】禁止在业务的更新类SQL语句中使用join，比如update t1 join t2…
3. 【强制】禁止使用子查询，建议将子查询SQL拆开结合程序多次查询。\(后续MySQL 8.0的子查询做了多级优化，后续升级可以用\)
4. 【强制】线上环境，多表join不要超过3个表，且不能为大表，大表的join性能开销极大，会造成数据库锁等待。\(bi分析库除外\)

- **事务**

1. 【建议】批量操作数据时，需要控制事务处理间隔时间，进行必要的sleep，一般建议值5\-10秒
2. 【建议】对于有auto\_increment属性字段的表的插入操作，并发需要控制在200以内。
3. 【强制】程序设计必须考虑“数据库事务隔离级别”带来的影响，包括脏读、不可重复读和幻读。线上建议事务隔离级别为repeatable\-read\(可重复读是适合支付，订单场景等需要。）其他的聊天，或者需要对锁不敏感的场景可以使用RC\)
4. 【建议】事务里包含SQL不超过5个（支付业务除外），因为过长的事务会导致锁数据较久，MySQL内部缓存、连接消耗过多等雪崩问题
5. 【建议】事务里更新语句尽量基于主键或unique key，如update … where id=XX;
6. 【建议】对于MySQL主从延迟严格敏感的select语句，请开启事务强制访问主库。

- **排序和分组**

1. 【建议】减少使用order by，和业务沟通能不排序就不排序，或将排序放到程序端去做。Order by、group by、distinct这些语句较为耗费CPU。
2. 【建议】order by、group by、distinct这些SQL尽量利用索引直接检索出排序好的数据。如where a=1 order by b可以利用key\(a,b\)。
3. 【建议】包含了order by、group by、distinct这些查询的语句，where条件过滤出来的结果集请保持在100行以内，否则SQL会很慢。

- **线上禁止使用的SQL语句**

1. 【强制】禁用procedure、function、trigger、views、event、外键约束。
2. 【强制】禁用insert into …on duplicate key update…在高并发环境下，会造成主从不一致。
3. 【强制】禁用replace 函数

- **线上数据库压测**

1. 【强制】新项目上线强制整体性能压测，压测生成量化报告，用数据评估业务系统到数据库的承载能力。
