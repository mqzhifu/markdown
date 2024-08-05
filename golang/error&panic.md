# 概览

大家认知中的异常处理：try catch 

GO的异常错误机制：error defer panic recover 

>GO 点反人类，虽然也看了文章，大概理解这么设计，但真是苦啊....

# error


已知错误，比如：函数如果正常执行，返回的是 nil，如果有错误即不为nil

优点：程序员永远手动处理，手动解决，都是已知的
缺点：程序员大师充斥着类似的判断，很恶心人

# panic


程序运行中，发现错误，如：空指针。
程序直接停止，报错

捕捉方法：defer + recover

这东西它只有在当前调用函数内使用，且还得是同一协程，且还得必须用 defer 配合用。
并不像 try catch ，在最上层监听一下，不管下面多少级的函数，只要出错，就直接能捕获。只需要写一次即可。

简单说就是：但凡你认为程序可能出错的地方就得有个 defer+recover 函数


所以 ，只要项目大一点，可能到处都是这种鬼东西。

# fatal

跟 panic 差不多，直接叫停程序，但不能捕获
>这个真是反人类中的反人类

1. all goroutines are asleep \- deadlock\!
2. concurrent map read and map write
3. concurrent map writes
4. runtime: out of memory
5. unexpected signal during runtime execution
6. stack overflow
7. found bad pointer in Go heap
8. signal SIGSEGV: segmentation violation

# defer

延迟函数/析构函数：函数执行完毕 ，会自动执行此函数

>recover 放在此 defer 函数内才生效，才能捕获 panic

#### 源码分析

```
type _defer struct {
  siz int32
  started bool
  heap bool
  openDefer bool
  sp uintptr // 栈指针
  pc uintptr // 调用方的程序计数器
  fn *funcval // 传入的函数
  _panic *_panic
  link *_defer // 指向下一个执行的defer函数
}
```

执行流程：创建

> defer func()

defer 为关键词，最终被解释成一个函数：runtime.deferproc

```go
func deferproc(siz int32, fn *funcval) { // arguments of fn follow fn
    //用户 goroutine 才能使用defer
    if getg().m.curg != getg() {
        // go code on the system stack can't defer
        throw("defer on system stack")
    }
    //也就是调用 deferproc 之前的 rs p寄存器的值
    sp := getcallersp()
    // argp 指向 defe r函数的第一个参数
    argp := uintptr(unsafe.Pointer(&fn)) + unsafe.Sizeof(fn)
    // 存储的是 caller 中，call deferproc 的下一条指令的地址
    // deferproc函数的返回地址
    callerpc := getcallerpc()

    //创建defer
    d := newdefer(siz)
    if d._panic != nil {
        throw("deferproc: d.panic != nil after newdefer")
    }
    //需要延迟执行的函数
    d.fn = fn
    //记录deferproc函数的返回地址
    d.pc = callerpc
    //调用deferproc之前rsp寄存器的值
    d.sp = sp
    switch siz {
	    case 0:
	        // Do nothing.
	    case sys.PtrSize:
	        *(*uintptr)(deferArgs(d)) = *(*uintptr)(unsafe.Pointer(argp))
	    default:
	        //通过memmove拷贝defered函数的参数
	        memmove(deferArgs(d), unsafe.Pointer(argp), uintptr(siz))
    }

    // deferproc通常都会返回0
    return0()
}
```

原理：
- 原函数的返回地址
- defer 函数的参数
- 下一个 defer 函数的入口地址
- 栈的地址，堆的初始化
- .......


```go
func newdefer(siz int32) *_defer {
  var d *_defer
  sc := deferclass(uintptr(siz))
  gp := getg() // 获得当前的 goroutine
  ...
  d.link = gp._defer // 现在新的 defer 函数的link指向了当前的defer函数
  gp._defer = d // 新的defer函数现在是第一个被调用的函数了
  return d
}
```

创建：\_defer 结构体，获取当前协程信息，将新的 \_defer结构体中的link成员变量，指向当前协程中的\_defer

```go
type _defer struct {
	siz     int32 // 参数和返回值的内存大小
	started bool
	heap    bool    // 区分该结构是在栈上分配的，还是对上分配的
	sp        uintptr  // sp 计数器值，栈指针；
	pc        uintptr  // pc 计数器值，程序计数器；
	fn        *funcval // defer 传入的函数地址，也就是延后执行的函数;
	_panic    *_panic  // panic that is running defer
	link      *_defer   // 链表,指向下一个 _defer
}
```

当正常函数执行完，开始执行该 g 的 defer 列表中的 defer 函数了，每个函数执行完后，就会走deferReturn

```go

func deferreturn(arg0 uintptr) {
  gp := getg() // 获得当前的goroutine
  d := gp._defer
  if d == nil { // 如果没有defer函数，直接return
    return
  }
  ...
  fn := d.fn // 获得 defer 的func函数
  d.fn = nil // 重置
  gp._defer = d.link // 将前一个defer函数 attach 到当前goroutine
  freedefer(d) // 释放 defer 函数
  _ = fn.fn // 执行defer的func函数
  jmpdefer(fn, uintptr(unsafe.Pointer(&arg0)))
```

获取当前协程信息，如果有 defer，则顺着当前协程信息，找到 \_defer连接队列，并判断是否属于此函数，然后从头部开始依次执行

源码分析：
1. 用户在某个函数中创建了一个 derfer func()
2. 编译器，遇到  derfer 关键字，开始解析
3. derfer 关键字会转换成  deferproc

执行分析过程：

1. G 结构体中，有一个成员变量：* \_defer
2. 当一个函数正常执行完毕，到 return 时
3. 先保存 return 的值，到一个临时变量中
4. 开始找 G的成员变量：  \_defer
5. 如果有，就开始执行  \_defer 对应的函数
6. 执行完一个 defer 函数后，判断后来是否还有 defer
7. 直到所有 derfer 函数执行完
8. 把第3步保存的临时值，返回



就是一个保存\<回调函数\>的链表（栈），该链表基于某个协程上。当函数执行 return 触发该链表的执行过程。return时 defer 执行顺序：后进先出


defer在一个函数中，可以定义N个。可以在任意位置定义。
当一个当时定义了多个 defer 函数，执行顺序：后进行出


单说 defer 的原理很简单，但是配合其它语法及使用场景，就麻烦了。

#### defer 与 return

return：
1. return 没有返回值，直接执行 defer 函数，比较简单
2. return 有返回值，但返回值没有定义变量名，都是些简单类型的值，新创建一个临时变量 ret，再将返回值保存于ret中
>注：返回值变量是指，函数的返回值的变量，如：func a ()(err errors,i int)
4. return 有返回值，并且返回值定义了变量名，会把此变量的指针地址传进 defer 函数
5. 检查是否有 defer，如果有就执行
6. 返回刚刚保存的临时变量

defer 居然是插在 return 执行过程中的一步。也就是说：return 是非原子操作：
- 保存 返回值
- 执行 defer
- 返回数据

>这里其实多了一步：执行 defer

实际例子：

```go
func test1() int {
	i := 1
	defer func() {   // 实际则是将局部变量i的地址指针传入，调用runtime.deferproc函数
		i++
	}()
	return i     // 将i的值拷贝到调用栈的返回值上
}

func test2() int {
	i := 1
	defer func(i int) {  // 将i的值拷贝一份传入,调用runtime.deferproc函数
		i++
	}(i)
	return i
}

func test3() (i int) { // 提前声明了返回值i
	i = 1
	defer func() { // 将返回值i的地址指针传入,调用runtime.deferproc函数
		i++
	}()
	return i  // 直接将值赋给调用栈的返回值上
}
```

简单说：defer 函数内部，使用了外部的变量。
如果是：值传递，正常。如果是地址传递，肯定会影响函数外面的变量。
这里  test3 定义了变量：i ，传给了 defer 就是地址传递。

#### defer 与 os.Exit

这是个无敌的东西，触发即立刻停止忽略一切，包括 defer

#### defer 与 panic

会触发 defer，且会所有 defer 函数均执行完，最后向外部程序报 panic
但在，执行 defer 过程中，遇到  recover ，那么，panic 消失。但后续的 defer 都是正常执行的

#### defer 与 fatal

一样，这个跟  exit 差不多，程序直接停止，defer 不会执行 recover 也捕获不到

#### 注意事项：

1. 变量作用域|影响了返回值
    地址|引用
    返回值为函数自带的变量名
    多 defer 函数同时修改父级变量
2. 受其它因素( exit | fata l return )影响，要知道是否能正确执行了defer
3. defer整体链是挂在一个协程中的
4. defer函数里：还可以再用协程，但还是尽量不用

应用场景：

1. lock/unlock
2. openFD/closeFD
3. 发生了运行时异常，容错处理

#### 异常捕获

- error  这个没太大影响
- panic 可以使用 defer 捕获
- fatal  无法捕获
# 总结

golang 并没有 try catch 这套东西，千万别把概念弄混了
defer 这东西有点复杂，最好做到：
- 不要引用外部变量，尤其是：返回值
- 只做一些收尾的工作，不要弄的太复杂
- 不要定义太多 defer  函数
- 只获取  panic
- 不要再开 子协程
