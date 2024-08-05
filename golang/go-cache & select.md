
# go-cache

go语言，自己实现一套内存缓存。
它没有具体 的实现，只是一组接口

```go
type StoreInterface interface {
    Get(key interface{}) (interface{}, error)//获取缓存
    Set(key interface{}, value interface{}, options *Options) error//设置缓存
    Delete(key interface{}) error//删除
    Invalidate(options InvalidateOptions) error
    Clear() error//清空
    GetType() string
}
```
>可能略有些差别，但大差不差


网上比较常用的几个实现库：go-cache  freecache, bigcache 

这里就以 go-cache  为DEMO吧，做分析

引入包：
>github.com/patrickmn/go-cache

```go
// 创建一个缓存对象，设置：超时时间 清理时间  
c := cache.New(5*time.Minute, 10*time.Minute)  
// 设置缓存值并带上过期时间  
c.Set("foo", "bar", cache.DefaultExpiration)  
// 设置没有过期时间的KEY，这个KEY不会被自动清除，想清除使用：c.Delete("baz")  
c.Set("baz", 42, cache.NoExpiration)  
  
c.Get("foo")  
c.Delete("baz")
```

使用起来挺简单的，看一下源码：

```go
package cache

// Item 每一个具体缓存值
type Item struct {
	Object     interface{}
	Expiration int64 // 过期时间:设置时间+缓存时长
}

// Cache 整体缓存
type Cache struct {
	*cache
}

// cache 整体缓存
type cache struct {
	defaultExpiration time.Duration // 默认超时时间
	items             map[string]Item // KV对
	mu                sync.RWMutex // 读写锁，在操作（增加，删除）缓存时使用
	onEvicted         func(string, interface{}) // 删除KEY时的CallBack函数
	janitor           *janitor // 定时清空缓存的结构
}

// janitor  定时清空缓存的结构
type janitor struct {
	Interval time.Duration // 多长时间扫描一次缓存
	stop     chan bool // 是否需要停止
}

```


感觉核心就是：读写锁+MAP
>go-cache 这个包还是挺简单的。

有个问题：读写锁更偏向于读多写少，一但MAP写过多，会导致竞争过多。或者 MAP 里的数据过多，清理程序与读写并发时，竞争也会变多。问题多多


具说 freecache, bigcache  更复杂一点，可能性能更好一点，我没继续研究


#### go-cache 与 redis memcache 区别 

提到 cache 机制，首先肯定想到的就是 redis memcache 这种开源稳定的内存缓存。
go-cache 原本就是单机/本机的，语言内部自己的缓存机制，但都会与开源软件对比与实现：
- go-cache 直接做成 redis memcache 模式，再加上分布式功能
- 为什么不直接使用  redis memcache 

我是觉得，GO-CACHE 挺鸡肋的


#### go-cache 与 map sync.map 区别

我是实在想不出：为毛要搞个GO-CACHE，sync.map 也是线程安全，且性能要更高。go-cache 只是多了一个定期功能，但一定要有这个功能嘛？自己删除不行嘛？我更倾向使用 使用 sync.map

另外，我看网上有拿 go-cache 实现了分布式缓存的~，呃。。。  直接用个分布式缓存的3方软件不就得了。。。  ，如果没那么大的项目，使用 redis 它不更好。如果只是小批量缓存点数据，直接上 sync.map不就得了。。。就感觉这个  go-cache 挺鸡肋。


# select

select 在编译的时候，会被转换成其它代码，其中有下面几种情况：
>defer 函数也差不多，遇到 defer 关键字会进行代码转换

1. 没有 case
2. 一个 case
3. 一个 case + default
4. 多个 case + defualt
5. 多个 case

没有 case 这种最简单，直接 gopark ，也可以理解为：死循环，并让出执行权
```go
func block() {
	gopark(nil, nil, waitReasonSelectNoCases, traceEvGoStop, 1)
}
```

>这种情况比较少，直接用 for 死循环是一样的。


一个 case 这种情况比较少，因为完全没必要用 select ，直接一行代码：   <- ch  ，自己阻塞不就得了
实际上编译 select 的时候，也是这么干的


一个 case + default:

```go
if selectnbsend(ch, i) {
    ...
} else {
    ...
}

func selectnbsend(c *hchan, elem unsafe.Pointer) (selected bool) {
	return chansend(c, elem, false, getcallerpc())
}

```


核心函数：selectnbsend-> chansend

chansend：是管道里面使用的，就是在 收发消息的时候，看是否有缓冲可IO数据，如果没有，不阻塞该协程，立刻返回



多个 case：找寻所有  chanel 中哪些可以IO的，有就执行，没有就，把自己注册成sudog ，发送给所有的chanel



多个 case + default : 不挂起，直接 走 default，重复循环


