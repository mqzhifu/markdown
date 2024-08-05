
# zval / 变量

PHP中变量类型：标量类型、复杂类型、特殊类型


|  |  |  |
|:---|:---|:---|
| 标量 | int string bool float |  |
| 复杂 | array object |  |
| 特殊 | 资源 和 null |  |  


所有的变量，最终编译成 ZEND opcode 的时候，都是申请一个 zval 结构体保存，结构如下：

```C
typedef struct _zval_struct {
	zvalue_value value; /* 储存变量的值 */
	zend_uint refcount__gc; /* 引用计数，用于GC */
	zend_uchar type; /* 储存变量类型 */
	zend_uchar is_ref__gc; /* 是否为引用，即 &$a */
} zval;
```

```c
//zvalue_value结构体
typedef union _zvalue_value {
    long lval; /* 长整型 */
    double dval; /* 浮点型 */
    struct {
        char *val;
        int len; /* 字符串的长度 */
    } str; /* 字符串 */
    HashTable *ht; /* 哈希 */
    zend_object_value obj; /* 对象 */
} zvalue_value;
```

>注：以上是PHP5的结构定义，PHP7中做了大幅改动，但因其复杂、且核心原理差不多，不写了

一个变量最终就是保存在 \_zval\_struct 结构体内，其中：zvalue_value 才是最终保存实际值的容器。

|  |  |  |
|:---|:---|:---|
| lval | 布尔、整形、资源 |  |
| dval | 浮点 |  |
| str | 字符串 |  |
| ht | 数组 |  |
| obj | 存储对象引用 |  |
| 所有成员变量均为：0 or null | null |  |  

zval 中的 type 字段，标识该变量的类型，如：

|  |  |  |
|:---|:---|:---|
| IS_LONG | 整形 |  |
| IS_BOOL | 布尔值 |  |
| IS_STRING | 字符串 |  |
| IS_ARRAY | 数组 |  |
| IS_RESOURCE | 文件资源\|obj  -> obj |  |  

通过  zval_value + type 标识 ，就实现了弱类型

#### DEMO：

$b= 3;

1、申请一个zval结构体
2、3是个整形，于是，zval.type = long
3、union结构体最后给了value变量，给变量赋值value.long = 3

字符串类型的会用 union 中的str保存

bool、资源、int 通用： lval存

数组:HashTable \*ht
对象：zend_object_value

神奇的\<弱类型\>就这样被完整的实现了。

# GC简要说明


#### 先要明白一点

PHP是解释型语言，一次执行后，释放所有内存
>反向看：PHP并不是守护进程的模式，对于内存的垃圾回收，并不是非常重要。就算有内存泄露，只要不是特别离谱，一次后必然全释放。

但，加上GC机制，未必全是好事儿，即使使用的是简单的引用计数，还是会影响正常代码的执行速度。
既然5.3加上了垃圾回收，也证明 PHP 还是可以写 守护进程 模式的
#### 垃圾回收-算法

引用计数法：当一个值，被引用次数为0时，此值在内存中即是垃圾了，可以清查
>计数法优点是简单，缺点是可能出现循环引用（内存泄露）
>PHP 就是使用此算法 做GC

#### zval 中的 GC 变量

每个变量最终是一个 zval ，其内部有两个成员变量：
- refcount__gc ：有多少个变量在引用
- is_ref__gc ：是否被引用，简单说，是否有使用&

例子：
```php
$a=1; //refcount__gc = 1,is_ref__gc = false
$b=$a;//refcount__gc = 2,is_ref__gc = false
```

```php
$a=1;//refcount__gc = 1,is_ref__gc = false
$b= &$a;//refcount__gc = 2,is_ref__gc = true
```


>实际上引擎是不会开辟两个内存空间来存的，除非其中一个变量值发生了变化，也就是写时复制，copy on wtrit.

unset 会触发 zend GC 进行检查，如果 refcount__gc = 0 ，会回收变量值
>unset 时：就是累减 refcount__gc

当 refcount__gc =0 时，GC会回收此变量
但，unset 不一定会触发 GC 删除机制，如果该变量还有其它引用，就不会删除

#### 循环引用


```php
$a = array( 'one');
$a[] =& $a;
unset($a);  
```


数组 $a 的第2个元素是自身的引用，成了 循环递归。无法被GC回收，内存泄漏。

# GC 详细

PHP5.3及以上版本，我们可以通过修改 php.ini 开启GC:

```php
zend.enable_gc = On
```


PHP内部定义了一个 zend_gc_globals 全局对象来管理GC：

```c
typedef struct _zend_gc_globals {
    zend_bool         gc_enabled; //是否启用
    zend_bool         gc_active; //是否处于正在运行状态

    gc_root_buffer   *buf;              /* preallocated arrays of buffers   */
    gc_root_buffer    roots;            /* list of possible roots of cycles */
    gc_root_buffer   *unused;           /* list of unused buffers           */
    gc_root_buffer   *first_unused;     /* pointer to first unused buffer   */
    gc_root_buffer   *last_unused;      /* pointer to last unused buffer    */

    zval_gc_info     *zval_to_free;     /* temporaryt list of zvals to free */
    zval_gc_info     *free_list;
    zval_gc_info     *next_to_free;

    zend_uint gc_runs; //gc_collect_cycles执行次数
    zend_uint collected; //缓冲池回收次数
	......
} zend_gc_globals;
```

两个重要成员变量：
- gc_root_buffer 
- zval_gc_info

创建一个新的变量，实际是如下的结构：

```c
typedef struct _zval_gc_info {
    zval z;
    union {
        gc_root_buffer       *buffered;
        struct _zval_gc_info *next;
    } u;
} zval_gc_info;
```

\*next 和 \*buffered 都是 null，只有被回收的时候，才会设置这两个值
\_gc_root_buffer  是个双向链表

```c
typedef struct _gc_root_buffer {
    struct _gc_root_buffer   *prev;        /* double-linked list               */
    struct _gc_root_buffer   *next;
    zend_object_handle        handle;    /* must be 0 for zval               */
    union {
        zval                 *pz;
        zend_object_handlers *handlers;
    } u;
} gc_root_buffer;
```


分析：
zend_gc_globals 实际内部是一个双向链表，节点类型为：\_gc_root_buffer
zval_gc_info 就是一个正常的变量，但当GC触发，会与  gc_root_buffer 进行双向绑定

当 unset 时，refcount__gc  - 1 ，同时触发GC机制：
1. 如果 ref = 0，直接删除此变量，解释内存
2. 如果 ref > 0 ，会把该变量放到  \_gc_root_buffer list 中
>节点标记成紫色
4. 触发GC机制，\_gc_root_buffer < 10000 ，直接 return
5. \_gc_root_buffer > 10000 ，遍历  zend_gc_globals中的所有节点
6. 深度遍历所有节点
7. 如果该节点是复杂类型，且内部还引用其它值，ref_cnt - 1，同时节点标记为灰色
8. 算法再次以深度优先判断每一个节点包含的zval的值，如果zval的refcount等于0，那么将其标记成白色\(代表垃圾\)，如果zval的 refcount大于0，那么将对此zval以及其包含的zval进行refcount加1操作，这个是对非垃圾的还原操作，同时将这些zval的颜色变 成黑色（zval的默认颜色属性）。
9. 遍历zval节点，将C中标记成白色的节点zval释放掉

模拟删除->模拟恢复->真正删除



>具体的删除逻辑我没看


php的GC回收机制启动时(php_module_startup),会分配10000个gc_root_buffer的空间：

```c
ZEND_API void gc_init(TSRMLS_D)  
{  
    if (GC_G(buf) == NULL && GC_G(gc_enabled)) {  
        GC_G(buf) = (gc_root_buffer*) malloc(sizeof(gc_root_buffer) * GC_ROOT_BUFFER_MAX_ENTRIES);  
        GC_G(last_unused) = &GC_G(buf)[GC_ROOT_BUFFER_MAX_ENTRIES];  
        gc_reset(TSRMLS_C);  
    }  
}
```


大概流程：
1. PHP启动时，会提前创建 10000 个GC节点
2. unset 时触发： zval 是否添加到  GC 节点中
3. 当 10000 个GC节点 都存了数据后，统一处理一次，删除垃圾

#### GC总结

咋说呢，它的GC触发机制比较延迟，且只有 unset + buf 容器大于 10000 。删除的算法也有点暴力。
应该是做了平衡聚会后，最终用了这个方法，也就是：PHP并不是纯后端语言，不是非常适合写守护进程的模式。更应该关心的是执行效率。

# 作用域

symble_table：保存全局变量，具体是，全局变量名和指向该变量名值的指针\(指向zval\)。

该结构体内还包含一个：active\_symble\_table，创建局部变量的时候，会往这个表里添加。

两个结构体，都是用hashTable结构存储。

实际上，我们每创建一个变量，最后都要加到symble\_table或active\_symble\_table。也就是说：一个函数内的所有变量是可以遍历的。

以上两个结构体保存于executor\_globals结构体中。

# 内存管理

主要工作就是维护三个列表：小块内存列表（free\_buckets）、大块内存列表（large\_free\_buckets）和剩余内存列表（rest\_buckets）。

ZEND提供了几个内存申请释放的接口，应用层在申请内存，如：定义一个变量，是通过这些接口向OS申请内存，但不是用多少申请多少，而是一次申请略大一点的内存，释放的时候，也不是直接向OS释放，而是把这部分内存放到剩余内存列表中去。

申请内存，逻辑如下：

![ae016389f34b7982f9004a7de0c44c55.jpg](image/ae016389f34b7982f9004a7de0c44c55.jpg)

![c6b456795c8a1a2c710feaf778efb5b9.jpg](image/c6b456795c8a1a2c710feaf778efb5b9.jpg)

先确定是大/小，如果是小，在小内存中，找不到，就去大中找，如果大找不到，就去空闲中找，如果还没有，就找OS要。

销毁变量

当执行unset 发现refcount=0，要归还内存的时候，就是把这段内存地址，放到rest\_buckets

实际发现：PHP 会先到OS，申请在大块内存，然后统计管理这块内存，这期间如果再有内存申请，直接从已申请的内存中操作，提升执行速度。缺点：一但某个请求执行时间过长，或者长驻内存，会导致大块内存长时间占用。但WEB开发就是一次性请求，然后全释放。
