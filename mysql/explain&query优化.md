# explain&query优化


```
+----+-------------+-------+------------+------+---------------+-----+---------+------+------+----------+-------+ | id | select_type | table | partitions | type | possible_keys | key | key_len | ref | rows | filtered | Extra | +----+-------------+-------+------------+------+---------------+-----+---------+------+------+----------+-------+
```


mysql\> explain select \* from user\\G

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\* 1. row \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

id: 1
select\_type: SIMPLE


table: user

type: ALL

possible\_keys: NULL

key: NULL

key\_len: NULL

ref: NULL

rows: 706011

Extra:

Table： 显示这一行的数据是关于哪张表

select\_type：

SIMPLE|简单查询，不使用UNION或子查询、联表

PRIMARY|子查询时，此表示为外层SELECT或UNION时左边的表

UNION|子查询时，此表示为内层SELECT或UNION时右边的表

DEPENDENT UNION|UNION中的第二个或后面的SELECT语句

UNION RESULT|UNION的结果

SUBQUERY|子查询中，内层SELECT

DEPENDENT SUBQUERY|子查询中的第一个SELECT，取决于外面的查询

DERIVED|派生表的SELECT FROM子句的子查询

Type：这是重要的列，显示连接使用了何种类型。从最好到最差的连接类型为system、const、eq\_reg、ref、range、indexhe和ALL

System|表只有一行：system表。这是const连接类型的特殊情况

const|表中的一个记录的最大值能够匹配这个查询（索引可以是主键或惟一索引）。因为只有一行，这个值实际就是常数，因为MYSQL先读这个值然后把它当做常数来对待

eq\_ref| 在连接中，MYSQL在查询时，从前面的表中，对每一个记录的联合都从表中读取一个记录，它在查询使用了索引为主键或惟一键的全部时使用

ref| 这个连接类型只有在查询使用了不是惟一或主键的键或者是这些类型的部分（比如，利用最左边前缀）时发生。对于之前的表的每一个行联合，全部记录都将从表中读出。这个类型严重依赖于根据索引匹配的记录多少—越少越好

range| 这个连接类型使用索引返回一个范围中的行，比如使用\>或\<查找东西时发生的情况

index| 这个连接类型对前面的表中的每一个记录联合进行完全扫描（比ALL更好，因为索引一般小于表数据）

ALL| 这个连接类型对于前面的每一个记录联合进行完全扫描，这一般比较糟糕，应该尽量避免

possible\_keys：可能应用在这张表中的索引。如果为空，没有可能的索引

key： 实际使用的索引。如果为NULL，则没有使用索引。很少的情况下，MYSQL会选择优化不足的索引。这种情况下，可以在SELECT语句中使用USE INDEX（indexname）来强制使用一个索引或者用IGNORE INDEX（indexname）来强制MYSQL忽略索引

key\_len： 使用的索引的长度。在不损失精确性的情况下，长度越短越好

ref： 显示索引的哪一列被使用了，如果可能的话，是一个常数

rows ：MYSQL认为必须检查的用来返回请求数据的行数

Extra：关于MYSQL如何解析查询的额外信息。最坏的例子是Using temporary和Using filesort，MYSQL

Distinct|一旦MYSQL找到了与行相联合匹配的行，就不再搜索了

Not exists|MYSQL优化了LEFT JOIN，一旦它找到了匹配LEFT JOIN标准的行，就不再搜索了

Range checked for each

Record（index map:\#）|没有找到理想的索引，因此对于从前面表中来的每一个行组合，MYSQL检查使用哪个索引，并用它来从表中返回行。这是使用索引的最慢的连接之一

Using filesort|看到这个的时候，查询就需要优化了。MYSQL需要进行额外的步骤来发现如何对返回的行排序。它根据连接类型以及存储排序键值和匹配条件的全部行的行指针来排序全部行

Using index|列数据是从仅仅使用了索引中的信息而没有读取实际的行动的表返回的，这发生在对表的全部的请求列都是同一个索引的部分的时候

Using temporary|看到这个的时候，查询需要优化了。这里，MYSQL需要创建一个临时表来存储结果，这通常发生在对不同的列集进行ORDER BY上，而不是GROUP BY上

Where used|使用了WHERE从句来限制哪些行将与下一张表匹配或者是返回给用户。如果不想返回表中的全部行，并且连接类型ALL或index，这就会发生，或者是查询有问题

mysql常用的hint

\[b\]强制索引 FORCE INDEX\[/b\]

SELECT \* FROM TABLE1 FORCE INDEX \(FIELD1\) …

以上的SQL语句只使用建立在FIELD1上的索引，而不使用其它字段上的索引。

\[b\]忽略索引 IGNORE INDEX\[/b\]

SELECT \* FROM TABLE1 IGNORE INDEX \(FIELD1, FIELD2\) …

在上面的SQL语句中，TABLE1表中FIELD1和FIELD2上的索引不被使用。

\[b\]关闭查询缓冲 SQL\_NO\_CACHE\[/b\]

SELECT SQL\_NO\_CACHE field1, field2 FROM TABLE1;

有一些SQL语句需要实时地查询数据，或者并不经常使用（可能一天就执行一两次）,这样就需要把缓冲关了,不管这条SQL语句是否被执行过，服务器都不会在缓冲区中查找，每次都会执行它。

\[b\]强制查询缓冲 SQL\_CACHE\[/b\]

SELECT SQL\_CALHE \* FROM TABLE1;

如果在my.ini中的query\_cache\_type设成2，这样只有在使用了SQL\_CACHE后，才使用查询缓冲。

\[b\]优先操作 HIGH\_PRIORITY\[/b\]

HIGH\_PRIORITY可以使用在select和insert操作中，让MYSQL知道，这个操作优先进行。

SELECT HIGH\_PRIORITY \* FROM TABLE1;

\[b\]滞后操作 LOW\_PRIORITY\[/b\]

LOW\_PRIORITY可以使用在insert和update操作中，让mysql知道，这个操作滞后。

update LOW\_PRIORITY table1 set field1= where field1= …

\[b\]延时插入 INSERT DELAYED\[/b\]

INSERT DELAYED INTO table1 set field1= …

INSERT DELAYED INTO，是客户端提交数据给MySQL，MySQL返回OK状态给客户端。而这是并不是已经将数据插入表，而是存储在内存里面等待排队。当mysql有 空余时，再插入。另一个重要的好处是，来自许多客户端的插入被集中在一起，并被编写入一个块。这比执行许多独立的插入要快很多。坏处是，不能返回自动递增 的ID，以及系统崩溃时，MySQL还没有来得及插入数据的话，这些数据将会丢失。

\[b\]强制连接顺序 STRAIGHT\_JOIN\[/b\]

SELECT TABLE1.FIELD1, TABLE2.FIELD2 FROM TABLE1 STRAIGHT\_JOIN TABLE2 WHERE …

由上面的SQL语句可知，通过STRAIGHT\_JOIN强迫MySQL按TABLE1、TABLE2的顺序连接表。如果你认为按自己的顺序比MySQL推荐的顺序进行连接的效率高的话，就可以通过STRAIGHT\_JOIN来确定连接顺序。

\[b\]强制使用临时表 SQL\_BUFFER\_RESULT\[/b\]

SELECT SQL\_BUFFER\_RESULT \* FROM TABLE1 WHERE …

当我们查询的结果集中的数据比较多时，可以通过SQL\_BUFFER\_RESULT.选项强制将结果集放到临时表中，这样就可以很快地释放MySQL的表锁（这样其它的SQL语句就可以对这些记录进行查询了），并且可以长时间地为客户端提供大记录集。

\[b\]分组使用临时表 SQL\_BIG\_RESULT和SQL\_SMALL\_RESULT\[/b\]

SELECT SQL\_BUFFER\_RESULT FIELD1, COUNT\(\*\) FROM TABLE1 GROUP BY FIELD1;

一般用于分组或DISTINCT关键字，这个选项通知MySQL，如果有必要，就将查询结果放到临时表中，甚至在临时表中进行排序。SQL\_SMALL\_RESULT比起SQL\_BIG\_RESULT差不多，很少使用。

二、MySQL对查询的自动优化

索引对于数据库是非常重要的。在查询时可以通过索引来提高性能。但有时使用索引反而会降低性能。

.

ID INT\(10\) UNSIGNED NOT NULL AUTO\_INCREMENT,

15. PRIMARY KEY\(ID\)，

17. INDEX \(NAME\)，

19. INDEX \(SALE\_DATE\)

假设这个表中保存了数百万条数据，而我们要查询商品号为1000的商品在2004年和2005年的平均价格。我们可以写如下的SQL语句:

SELECT AVG\(PRICE\) FROM SALES

WHERE ID = 1000 AND SALE\_DATE BETWEEN '2004\-01\-01' AND '2005\-12\-31';

如果这种商品的数量非常多，差不多占了SALES表的记录的50%或更多。那么使用SALE\_DATE字段上索引来计算平均数就有些慢。因为如果使 用索引，就得对索引进行排序操作。当满足条件的记录非常多时\(如占整个表的记录的50%或更多的比例\)，速度会变慢，这样还不如对整个表进行扫描。因 此，MySQL会自动根据满足条件的数据占整个表的数据的比例自动决定是否使用索引进行查询。

对于MySQL来说，上述的查询结果占整个表的记录的比例是30%左右时就不使用索引了，这个比例是MySQL的开发人员根据他们的经验得出的。然而，实际的比例值会根据所使用的数据库引擎不同而不同。

三、 基于索引的排序

MySQL的弱点之一是它的排序。虽然MySQL可以在1秒中查询大约15,000条记录，但由于MySQL在查询时最多只能使用一个索引。因此，如果WHERE条件已经占用了索引，那么在排序中就不使用索引了，这将大大降低查询的速度。我们可以看看如下的SQL语句:

1. SELECT \* FROM SALES WHERE NAME = “name” ORDER BY SALE\_DATE DESC;
    在以上的SQL的WHERE子句中已经使用了NAME字段上的索引，因此，在对SALE\_DATE进行排序时将不再使用索引。为了解决这个问题，我们可以对SALES表建立复合索引:
2. ALTER TABLE SALES DROP INDEX NAME, ADD INDEX \(NAME, SALE\_DATE\)
    这样再使用上述的SELECT语句进行查询时速度就会大副提升。但要注意，在使用这个方法时，要确保WHERE子句中没有排序字段，在上例中就是不能用SALE\_DATE进行查询，否则虽然排序快了，但是SALE\_DATE字段上没有单独的索引，因此查询又会慢下来。

五、 使用各种查询选择来提高性能

1. STRAIGHT\_JOIN:强制连接顺序
    当我们将两个或多个表连接起来进行查询时，我们并不用关心MySQL先连哪个表，后连哪个表。而这一切都是由MySQL内部通过一系列的计算、评估，最后得出的一个连接顺序决定的。如下列的SQL语句中，TABLE1和TABLE2并不一定是谁连接谁:
2. SELECT TABLE1.FIELD1, TABLE2.FIELD2 FROM TABLE1 ,TABLE2 WHERE …
    如果开发人员需要人为地干预连接的顺序，就得使用STRAIGHT\_JOIN关键字，如下列的SQL语句:
3. SELECT TABLE1.FIELD1, TABLE2.FIELD2 FROM TABLE1 STRAIGHT\_JOIN TABLE2 WHERE …
    由上面的SQL语句可知，通过STRAIGHT\_JOIN强迫MySQL按TABLE1、TABLE2的顺序连接表。如果你认为按自己的顺序比MySQL推荐的顺序进行连接的效率高的话，就可以通过STRAIGHT\_JOIN来确定连接顺序。
4. 干预索引使用，提高性能
    在上面已经提到了索引的使用。一般情况下，在查询时MySQL将自己决定是否使用索引，使用哪一个索引。但在一些特殊情况下，我们希望MySQL只使用一个或几个索引，或者不希望使用某个索引。这就需要使用MySQL的控制索引的一些查询选项。
    限制使用索引的范围
    有时我们在数据表里建立了很多索引，当MySQL对索引进行选择时，这些索引都在考虑的范围内。但有时我们希望MySQL只考虑几个索引，而不是全部的索引，这就需要用到USE INDEX对查询语句进行设置。
5. SELECT \* FROM TABLE1 USE INDEX \(FIELD1, FIELD2\) …
    从以上SQL语句可以看出，无论在TABLE1中已经建立了多少个索引，MySQL在选择索引时，只考虑在FIELD1和FIELD2上建立的索引。
    限制不使用索引的范围
    如果我们要考虑的索引很多，而不被使用的索引又很少时，可以使用IGNORE INDEX进行反向选取。在上面的例子中是选择被考虑的索引，而使用IGNORE INDEX是选择不被考虑的索引。
6. SELECT \* FROM TABLE1 IGNORE INDEX \(FIELD1, FIELD2\) …
    在上面的SQL语句中，TABLE1表中只有FIELD1和FIELD2上的索引不被使用。
    强迫使用某一个索引
    上面的两个例子都是给MySQL提供一个选择，也就是说MySQL并不一定要使用这些索引。而有时我们希望MySQL必须要使用某一个索引\(由于 MySQL在查询时只能使用一个索引，因此只能强迫MySQL使用一个索引\)。这就需要使用FORCE INDEX来完成这个功能。
7. SELECT \* FROM TABLE1 FORCE INDEX \(FIELD1\) …
    以上的SQL语句只使用建立在FIELD1上的索引，而不使用其它字段上的索引。
8. 使用临时表提供查询性能
    当我们查询的结果集中的数据比较多时，可以通过SQL\_BUFFER\_RESULT.选项强制将结果集放到临时表中，这样就可以很快地释放MySQL的表锁\(这样其它的SQL语句就可以对这些记录进行查询了\)，并且可以长时间地为客户端提供大记录集。
9. SELECT SQL\_BUFFER\_RESULT \* FROM TABLE1 WHERE …
    和SQL\_BUFFER\_RESULT.选项类似的还有SQL\_BIG\_RESULT，这个选项一般用于分组或DISTINCT关键字，这个选项通知MySQL，如果有必要，就将查询结果放到临时表中，甚至在临时表中进行排序。
10. SELECT SQL\_BUFFER\_RESULT FIELD1, COUNT\(\*\) FROM TABLE1 GROUP BY FIELD1
