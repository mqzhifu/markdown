# 数组

一个容器，在内存是一段连续的空间
>跟其它语言都差不多

创建一个数组：
```go
var a [3]int
a := [3]int{1,2,3}
```

GO 函数的参数，都是值传递。数组也是值传递
>函数的参数为数组类型，函数内部改动数组里的值，外部的数组值不变

```go
func main() {
	arrayA := [2]int{100, 200}
	var arrayB [2]int

	arrayB = arrayA

	fmt.Printf("arrayA : %p , %v\n", &arrayA, arrayA)
	fmt.Printf("arrayB : %p , %v\n", &arrayB, arrayB)

	testArray(arrayA)
}

func testArray(x [2]int) {
	fmt.Printf("func Array : %p , %v\n", &x, x)
}
```

>结果是3个不同的地址

从上面看，如果一个函数的参数是数组类型，且数组较大，那值复制操作也很大。改成地址传参：

可以把函数的参数类型改成，地址引用：  

```go
func t2(x *[2]int) {  
	fmt.Printf("func Array : %p , %v\n", &x, x)  
}
t2(&arrayA)
```

这样，函数内部修改了数据组的值，函数外面的数组也会跟着改变

但数组做为参数，有个很烦的问题：数组大小必须统一 ，如：参数定义数组容量为 2，你传的就得是2。
# 切片 Slices


#### 结构体

```go
type slice struct {
	array unsafe.Pointer //指针。指向一个地址(数组首部)
	len   int //数组的长度
	cap   int //数组可容纳的元素
}
```

在传统 数组上又封装了一层，增加了两个属性：len 和 cap

正常一个数组，在静态语言中，肯定是定长的
如果想要动态变长(动态扩容)，就得有一个简单的算法，实现动态扩容，Slices 就可以。

```go
var []int a      //注：不能设定切片大小，否则就跟数组一样了
a := []int{1,2,3}//注：不能设定切片大小，否则就跟数组一样了
//make( type , len ,cap )
a := make([]int,3,6)
a := make([]int,4)
//从数组中创建切片
arar := [4]{1,2,3,4}
a := [1:3] // 结果：2 3，从第1个(数组从0开始)位置开始截取，到第3个位置结束   [1:3)       
```

# 什么情况会产生扩容

给数组添加元素：append / copy 函数

#### 扩容策略：
1. 如果新申请的容量，是原容量的 2 倍，直接扩容
2. 如果旧切片的长度小于1024，则最终容量(newcap)就是旧容量(old.cap)的两倍、
3. 如果旧切片长度大于等于1024，则最终容量（newcap）从旧容量（old.cap）开始循环增加原来的 1/4
4. 如果最终容量（cap）计算值溢出，则最终容量（cap）就是新申请容量

#### 函数参数

GO 里，函数参数肯定都是值传递(指定参数为引用除外)，那么，slice 也是值传递。
它会复制 slice 结构体，但是它是浅 copy，即：slice 结构体中的成员变量 array ，这个实际还是指向原数组的地址。
也就是：slice 做为函数参数，是地址传递。即：函数内部修改了 slice 的值，外部也会修改

有种特殊的情况，当使用 append 给元素添加内容时，产生了扩容操作：新申请了一块内存，将把slice 内部的 array 成员变量的指针，指向了一块新的内存区域。那么：函数内部修改 slice 并不会影响外部


#### 注意事项

切片是很灵活：
- 能自动扩容
- 可以当成函数参数的地址引用 
- 元素截取

在给函数做为参数时，操作一定要小心 。尤其使用函数：append 和 copy ，可能造成函数内部修改了，但外部没修改的情况 。

说到底，它是浅 copy。一但产生分裂效果，外部的值就不会跟着修改。


扩容过程中，可能会产生bug?

 ```
 a := [4]int{1,2,3,4}
 b := a[1,3]
 c = append(1,b)
 ```



这里的问题是出在 append ，返回的是一个新的数组，还是旧的


#### 元素删除

没有专门的删除函数，可以通过元素截取
```
a := []int{1,2,3}
b := a[1:] //删除第一个元素
c := a[0:len(a)- 1] //删除最后一个元素
```
#### 总结

感觉切片的模式，是弥补了数组的扩展性，避免了因为定长，而使开发者频繁 的需要手动扩容数组。还可以动态切割数组元素。那么使用的结论就是：数组 + 切片配合使用。

# MAP

```go
type hmap struct {
    count     int // 元素的个数
	// 1. key和value是否包指针
    // 2. 是否正在扩容
    // 3. 是否是同样大小的扩容
    // 4. 是否正在 `range`方式访问当前的buckets
    // 5. 是否有 `range`方式访问旧的 bucket
    flags     uint8 // 标记读写状态，主要是做竞态检测，避免并发读写
    B         uint8  // 可以容纳 2 ^ N 个 bucket
    noverflow uint16 // 溢出的bucket个数
    hash0     uint32 // hash 因子

    buckets    unsafe.Pointer // 指向数组 buckets 的指针
    oldbuckets unsafe.Pointer // growing 时保存原 buckets 的指针
    nevacuate  uintptr        // growing 时已迁移的个数

    extra *mapextra
}

type mapextra struct {
    overflow    *[]*bmap // 指针数组，指向所有溢出桶 oldoverflow *[]*bmap
    oldoverflow *[]*bmap // 指针数组，发生扩容时，指向所有旧的溢出桶 nextOverflow *bmap

    nextOverflow *bmap // 指向 所有溢出桶中 下一个可以使用的溢出桶
}
//很简单，但是编译的时候，会生成的结构
type bmap struct {
     tophash [bucketCnt]uint8
}​
 //在编译期间会产生新的结构体
type bmap struct {
	tophash     [8]uint8 //存储哈希值的高8位
	keys        [8]keytype
	values      [8]valuetype
	pad         uintptr
	overflow    uintptr
	overflow    *bmap   //溢出bucket的地址
}
```

hmap 是一个汇总体：

|            |                                           |     |
| :--------- | :---------------------------------------- | :-- |
| count      | 元素总个数                                     |     |
| buckets    | 指针。指向一个数组：[]bmap, 存储 bmap（bucket ）        |     |
| bmap       | 具体数据的存储是，一个 bmap 里存8个KV。                  |     |
| B          | buckets 数组的长度的对数，也就是说 buckets 数组的长度就是 2^B |     |
| flags      | 当前 MAP 的状态。防止并发写的，他类似于全局锁。 出现就报 pnaic 了   |     |
| oldbuckets | 扩容时，存放之前的 buckets (Map扩容相关字段)             |     |
| extra      | 溢出桶结构，正常桶里面某个 bmap 存满了，会使用这里面的内存空间存放键值对   |     |
| noverflow  | 溢出桶里 bmap 大致的数量                           |     |

bmap 就是具体存储KV值的容器，一次存储8个KV：

|          |                                                                                              |
| -------- | -------------------------------------------------------------------------------------------- |
| tophash  | 长度为8的数组，元素为：map key 获取的 hash 的高8位，遍历时对比使用，提高性能                                               |
| keys     | 长度为8的数组，存储 key                                                                               |
| values   | 长度为8的数组，存储 value                                                                             |
| pad      | 对齐内存使用的，不是每个 bmap 都有会这个字段，需要满足一定条件                                                           |
| overflow | 指向的 hmap.extra.overflow 溢出桶里的 bmap，上面的字段 tophash、keys、elems 长度为8，最多存8组键值对，存满了就往指向的这个 bmap 里存 |

mapextra：

| 字段           | 解释                                                                                                                                                                         |
| ------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| overflow     | 称之为**溢出桶**。和`hmap.buckets`的类型一样也是数组`[]bmap`，当正常桶`bmap`存满了的时候就使用`hmap.extra.overflow`的`bmap`。所以这里有个问题正常桶`hmap.buckets`里的`bmap`是怎么关联上溢出桶`hmap.extra.overflow`的`bmap`呢？我们下面说。 |
| oldoverflow  | 扩容时存放之前的overflow(Map扩容相关字段)                                                                                                                                                |
| nextoverflow | 指向溢出桶里下一个可以使用的`bmap`                                                                                                                                                       |
![[golang-map.png]]
## hash 函数选择

在程序启动时，会检测 cpu 是否支持 aes，如果支持，则使用 aes hash，否则使用 memhash
## key hash 计算

将 KEY 转化成一个 数字：

1. 低 8 位用来确定：buckets 数组中，第几个元素(bmap)
2. 高 8 位指向具体某一个  bmap 中的真实的 key
>tohash 就存高8位的值

桶串联实现拉链法：当某个桶数量满了，会申请一个新桶，挂在这个桶后面形成链表，新桶优先使用预分配的桶。


>与传统hashTable对比：它多了一个二级 hash 结构（存储8个元素的桶）

一级是汇总数组的结构体( hmap )，然后指向一个数组 (buckets)，元素值：hmap-bucket，bucket 结构体里又是一个汇总结构，此结构体里又包含一个数组.（bmap）

hmap->arr-buckets -> bmap

## 流程

1. 创建一个MAP，即会有一个 hmap 结构体
    hmap结构体进行初始化结构体的元素值
    1. B值，这个是确定有几个bucket = 0
    2. buckets:指针，指向一个数组，这里是新创建一个数组
    3. old\-buckets:指针，指向一个数组，扩容时使用
    4. hash因子
    5. count = 0
    6. flags //处理并发写
    7. 溢出桶的一些信息
2. 此时，有一个元素添加
3. 将该元素的 KEY，HASH FUNC + hash 因子 = 转换成一个数字
4. 低8位再转换一个数字，数字范围:0~数组最大长度\-1，即得到数组的key
    准确的说是：低8位与 hmap中的B值做\<与\>运算
5. 此时，创建一个bucket，同时再创建3个数组
    keys values tophash
6. 将真实的 KV 值，存到这个3个新的数组中
7. 最后，将该 bucket 放置到第4步中得出K值的数组中
8. 将hmap中的count值\+1

## 查找

1. 将该元素的KEY，HASH FUNC + has因子 = 转换成一个数字
2. 根据低8位，与B做与运算，找到该bucket所在数组的索引位置
3. 用高8位循环比对：从该 bucket 结构的元素 tophash 数组中，查找
4. 如果找不到，再去 overflow 中找
5. 如果还是找不到，即该KEY不存在

这里的 tophash 有点意思，不是直接对真的 key 值比对，而是先找 tophash，然后如果找到了，根据索引值，直接去另外两个数组里取值。也算是个取巧吧，对KEY值的搜索优化

## 扩容/链表

触发扩容条件：

1. map 中实际存放元素总个数 / bucket > 6.5
2. overflow bucket 过多
    1. B \<= 15 ，已使用溢出桶个数\>= 2^B
    2. b \> 15 ,已使用溢出桶个数\>= 2^15

溢出扩容：是等量扩容
总个数不够：翻倍扩容

当触发扩容后，会新创建一个数组，且旧数组依然在保存，也就是新/旧两个桶会并行一段时间。如果此时有新的数据过来了，肯定是往新桶里添加。

数据迁移 ：并不是一次性迁移，而是在写入操作时，再迁移。
或者说：就算扩容出一个新的b-array，也是空的，新的数据会进来，旧的数据只有写操作时候才迁移 。两个b-array 会同时运行

## 线程安全

普通的一个map 是非线程安全的，即：读写共同操作MAP，就会报错：

同时有2个协程并发写：concurrent map read
读写并发：concurrent map read and map write
>操蛋的点也在这儿，允许并发读，不允许并发写、并发写读


加锁：
- 排它锁：性能太低
- 读写锁：能满足 MAP 线程安全

# sync.Map


普通的 map ，其内部有一个 flag 字段，一但当前有写操作，你再想写，就会报错。简单说：线程安全，但并发性能有点低。

读写锁能解决基本的线程安全，但是 G 过多的时候，会导致性能下降 

这里就引出了：sync.Map

```go
type Map struct {
	mu Mutex 
	read atomic.Value // readOnly 
	dirty map[interface{}]*entry 
	misses int 
}
```

```go
type readOnly struct { 
	m map[interface{}]*entry 
	amended bool 
}
```

```go
type entry struct { 
//是个指针
//read 和 dirty存在冗余情况（amended=false），但由于是指针类型，存储的空间应该不是问题 
	p unsafe.Pointer // *interface{} 
}
```


|        |                        |                                              |
| :----- | :--------------------- | :------------------------------------------- |
| mu     | Mutex                  | 排它锁。保护后文的 dirty 字段                           |
| read   | atomic.Value           | 存读的数据。并发是安全(atomic.Value)实际存的是readOnly的数据结构。 |
| misses | int                    | 计数作用。每次从read中读失败，则计数+1。                      |
| dirty  | map[interface{}]*entry | 包含最新写入的数据。当misses计数达到一定值，将其赋值给read。          |

|         |                        |                                     |
| ------- | ---------------------- | ----------------------------------- |
| m       | map[interface{}]*entry | 单纯的map结构                            |
| amended | bool                   | Map.dirty的数据和这里的 m 中的数据不一样的时候，为true |


一共就3个结构体，看着挺简单。核心就是使用了两个 map：
1. 专门负责读操作的 map
2. 专门负责写操作的 map

写 map 肯定比 读 map 的元素多(写 map 包含 读 map)
读操作时：先读取 (读 map)，如果不存在，再去( 写map )中读

当读操作时 (读map)未命中，而在(写map)中命中，会标记一次：未命中，当未命中数> (写map)元素个数时触发 ： (写map)  覆盖   (读map)
 
优点：是官方出的，是亲儿子；通过读写分离，降低锁时间来提高效率
缺点：不适用于大量写的场景，这样会导致 read map 读不到数据而进一步加锁读取，同时dirty map也会一直晋升为 read map，整体性能较差。 适用场景：大量读，少量写