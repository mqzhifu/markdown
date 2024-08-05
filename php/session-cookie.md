
# SESSION

#### 工作原理：

S 端建立 SESSION，以 HASH 散列保存于S端硬盘上，文件中包含SESSION的一些信息，然后，将SESSION\_ID，以COOKIE方式，响应回给浏览器。如果C端禁用了COOKIE,则在URL中传递。


#### 具体函数：

session\_start：开始一个会话 或 创建一个会话
>它有一定概率开启垃圾回收，同时，对应的session文件是被锁定的，直到当前脚本结束才会解锁。

session\_destroy：删除 SESSION 文件

session\_unset：删除内存中的session信息

#### 具体配置信息

session.use\_trans\_sid=1 : 是否开启url传递PHPSESSION ID

session.use\_only\_cookies=0 : 是否只允许使用Cookie 传递PHPSESSION ID

session.use\_cookies=1 : 是否开启Cookie传递PHPSESSION ID

session.gc\_probability =1

session.gc\_divisor =1000

session.gc\_maxlifetime =1440 : 过期时间 默认24分钟

>概率是 session.gc\_probability/session.gc\_divisor 结果 1/1000,
不建议设置过小，因为session的垃圾回收，是需要检查每个文件是否过期的。

session.save\_path : session 文件的存储位置
>这是随机分级存储，这个样的话，垃圾回收将不起作用，需要自己写脚本



销毁 ，单执行session\_destroy\(\)可以删除文件，但是当前执行脚本文件$\_SESSION还可以再用，如果单执行session\_unset\(\)，但文件还存在。就算以上两个一起执行，但COOKIE里还存着SESSION\-ID,也需要一并清理掉。

#### session 问题：

- 单服务器，以文件形式存于磁盘，增删改查，会增加 IO，可以放到内存或者DB中
- 多服务器共享SESSION，文件形势也有问题，同样内存DB中

#### session 存 DB

session.save\_handler = files //此处也可以换成use 就是 mysql

既然不用PHP自己的SESSION机制，那就得手动写代码了

session\_set\_save\_handler\( "open","close","read","write","destroy","gc" \);

先要注册几个着急函数

open

close

read

write

destory

gc

然后，新建信文件，写入类名，把上面的方法实现

接着创建DB表结构

使用 MySQL 保存 session，需要保存三个关键性的数据：session id、session数据、session生命期。

SESS存REDIS

session.save\_handler = redis

session.save\_path = "tcp://127.0.0.1:6379"

# COOKIE

Setcookie\(string name, string value, int expire,string path, string domain, int secure\);

C端请求，S端将数据附代于头中返回C端，C将信息保存下来。下次再将请求，C端把保存的数据取出来，附于请求头。

域名参数：可以设置主域名或者子域名如: baidu.com 、login.baidu.com

子域名可以访问顶级域名的COOKIE数据，但是同级不能访问，也不能修改顶级、同级域名的数据

顶级域名只能访问顶级域名的数据

COOKIE\+SESSION

目前有种方法是，登陆后，把UID SESSION\_id 保存到COOKIE中，加密。好处是，长时间保持连接，一但服务端接收到COOKIE中有UID 就创建SESSION。

单点登陆

同域名下，都把COOKIE设置成'/'，或者主域名

非同域下，登陆成功后，回带JS代码，让JS代码再访问其它域登陆，然后成功后，写入C端COOKIE中
