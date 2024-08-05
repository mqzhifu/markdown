原子操作。

以前只知道软件上加锁做原子操作。但其实在硬件上(CPU)，也可以实现原子操作。
 具体：当某核 CPU 要操作某块内存时，给总线发信号，其它信号如果想写内存，就会被阻塞。直到该CPU解锁后，其它进程才能写内存。
>intel 是给总线发信号，AMD是 MESI一致性协议

>这里其实我不是太明白，还牵扯到CPU的缓存知识，只知道个大概

# CAS

在高并发编程中，经常会出现对同一个资源并发访问修改的情况，为了保证最终结果的正确性，一般会使用 锁 和 CAS原子操作 来实现。
>如：计数统计

锁  和 CAS 的性能：大概比锁快了一丢丢 。个人觉得：快在CPU直接加锁，比在代码层面加锁能快一点点

# atomic  类

golang 里的原子操作，其实没多少代码，大多被转换成了汇编语言，主要是根据本机OS的的CPU来确定汇编代码。

包里大概有如下的方法：

```go
func AddInt32(addr *int32, delta int32) (new int32)
func AddInt64(addr *int64, delta int64) (new int64)
func AddUint32(addr *uint32, delta uint32) (new uint32)
func AddUint64(addr *uint64, delta uint64) (new uint64)
func AddUintptr(addr *uintptr, delta uintptr) (new uintptr)
```

```go
func CompareAndSwapInt32(addr *int32, old, new int32) (swapped bool)
func CompareAndSwapInt64(addr *int64, old, new int64) (swapped bool)
func CompareAndSwapUint32(addr *uint32, old, new uint32) (swapped bool)
func CompareAndSwapUint64(addr *uint64, old, new uint64) (swapped bool)
func CompareAndSwapUintptr(addr *uintptr, old, new uintptr) (swapped bool)
func CompareAndSwapPointer(addr *unsafe.Pointer, old, new unsafe.Pointer) (swapped bool)
```

```go
func SwapInt32(addr *int32, new int32) (old int32)
func SwapInt64(addr *int64, new int64) (old int64)
func SwapUint32(addr *uint32, new uint32) (old uint32)
func SwapUint64(addr *uint64, new uint64) (old uint64)
func SwapUintptr(addr *uintptr, new uintptr) (old uintptr)
func SwapPointer(addr *unsafe.Pointer, new unsafe.Pointer) (old unsafe.Pointer)

```

```go
func LoadInt32(addr *int32) (val int32)
func LoadInt64(addr *int64) (val int64)
func LoadUint32(addr *uint32) (val uint32)
func LoadUint64(addr *uint64) (val uint64)
func LoadUintptr(addr *uintptr) (val uintptr)
func LoadPointer(addr *unsafe.Pointer) (val unsafe.Pointer)

```

```go
func StoreInt32(addr *int32, val int32)
func StoreInt64(addr *int64, val int64)
func StoreUint32(addr *uint32, val uint32)
func StoreUint64(addr *uint64, val uint64)
func StoreUintptr(addr *uintptr, val uintptr)
func StorePointer(addr *unsafe.Pointer, val unsafe.Pointer)
```


根据前缀进行分类与分析：

| 函数名前缀          | 描述                                                                   |     |     |
| :------------- | :------------------------------------------------------------------- | :-- | --- |
| Add            | 对一个整型的变量进行加法操作，并返回新的值                                                |     |     |
| CompareAndSwap | 比较并交换一个指针型的变量的值。如果变量的值等于旧值，就将变量的值设置为新值，并返回 true；否则，不修改变量的值，并返回false。 |     |     |
| Load           | 获取一个指针型的变量的值                                                         |     |     |
| Store          | 设置一个指针型的变量的值                                                         |     |     |
| Swap           | 交换一个指针型的变量的值，并返回旧值                                                   |     |     |

感觉挺简单的，一个函数调用就完成了。参数大多是指针类型，需要注意下即可。


其它的使用方法：


```go
type Value struct {
	v any
}
// 加载Value中存的值
func (v *Value) Load() (val any) {}
// 存储对象放入Value中
func (v *Value) Store(val any) {}
// 交换Value中存的数据
func (v *Value) Swap(new any) (old any) {)
// 比较并存储（值必须铜类型且可比较采用使用该方法，否则panic）；传入old、new对象，如果old对象等于Value内存储的对象，就将新的对象存入，返回treu。否则false
func (v *Value) CompareAndSwap(old, new any) (swapped bool) {}

```

结构好像很简单，定义了一个 Value 类型，包含一个 V 变量，且什么类型都可以存储。
在 Value 上挂4个常用函数，使用起来更规范些。




