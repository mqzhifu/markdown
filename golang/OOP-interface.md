
# 继承

GO 里严格来说就没<继承>这个东西，而是通过匿名成员变量间接实现了大家认知里的：继承
>没有 extends 关键字

```go

type Animal struct {
	Name string
	Category int
}

type Rabbit struct {
	Animal
	Tail string
	Category int//这里定义了一个跟Animal一样的成员变量名

}

```

Rabbit 结构 继承了 Animal 结构

同时 Rabbit 和 Animal 还共有一个相同名称的成员变量：Category，且互不影响...

>rabbit.Category
>rabbit.Animal.Category

继承的作用？

1. 简少重复写代码
2. 统一入口、统一管理（父类的构造函数为公共的）

>想了很久，现实应用中就这么2个作用....
# 构造函数

go 没有标准的构造函数，需要程序员自己编写

```go
func NewAnimal(name string){  
	animal := new Animal()  
	animal.Name = name  
}
```

# 封装/class/成员函数/成员变量

GO 里并没有封装这个概念，没有 class 关键字，也没有大括号，括号里面的函数就是此class的方法。
它的所有封装操作都是基于 结构体的


```go
type Animal struct {
	Name string
}

func(animal *Animal)Eat(something string){
	.......
}
```

在函数的有前面，定义：挂载到某个结构体下面，eat 函数就是Animal 的成员函数
Name 就是 Animal 的成员变量


# 函数重写

```go

func (animal *Animal)Eat(){

	fmt.Println("im Animal Eat")

}

func (rabbit *Rabbit)Eat(){
	fmt.Println("im rabbitKid Eat")
}

```

Rabbit 和 Animal 同时还拥有一个相同的函数： Eat()

依然互不影响

子结构也可以同时继承多个父结构体，但是多个父结构体之间有相同名字的函数时，子结构调用的时候得加上标识，如：标识A结构体.函数1,如果直接调用，会报错。

从JAVA角度看：也可以间接理解JAVA里为啥不允许多继承，从GO的角度看：≈JAVA的OOP那套体系被GO这种实现方式直接给扒个精光....



# interface 多态

go 没有 implament 关键字，而且也没有和结构体绑定，它的接口实现可以算得上：飘逸！

定义一个接口，只要实现了此接口的所有方法，就算实现了些接口
>太魔幻了......   估计 是 go 引擎在编译的时候，判定接口是否被实现

```go
type Animal interface {  
	eat(some string)
}  
  
type Rabbit struct {  
}  
  
func (rabbit *Rabbit) Eat(some string) {  
  
}
```

多态感觉是个被高估的概念，在我看来：它就是定义一个接口，更像是一种协议，预先定义好。大家自行实现它。无奈，被神化了，反倒觉得 GO 里，更实际些。不光多态，还有继承 封装，GO都非常的简单 

# interface 数据类型

定义一个空接口（内部无任何函数名）

```go
type Animal interface{}
var a Animal
a = 1
a = "imstr"
```

一个变量即可以是整形又可以是字符串... 这就是GO里接口的强大用法。

```go
func PPP(x interface{})
```

定义一个函数，参数x为<空接口>类型，即：该函数可以接收任意类型的参数...

再变化一下

```go
func PPP(x ...interface{})
```

函数可以接收N个参数，且任意类型....这就是传说中的：多态

现实中大概率不会这么用：
1.  虽然你可以传进来任意类型，但是你使用的时候得先断言
2. 毕竟是强类型语言，传进来的东西还得再判断

>他不会给你真的提供弱类型语言特性，只是加了一点便捷而以....


#### 类型

interface 在运行时才会知道类型

1. interface 为空
2. interface 包含若干方法

runtime.iface：带方法的接口
runtime.eface：表示不带任何方法的空接口 interface{}

>这里主要讲，空 interface


```go
// $GOROOT/src/runtime/runtime2.go
type eface struct { // 16 字节
	_type *_type
	data  unsafe.Pointer
}
```



```go
//src/runtime/type.go
type _type struct {
	size       uintptr
	ptrdata    uintptr // size of memory prefix holding all pointers
	hash       uint32
	tflag      tflag
	align      uint8
	fieldAlign uint8
	kind       uint8
	// function for comparing objects of this type
	// (ptr to object A, ptr to object B) -> ==?
	equal func(unsafe.Pointer, unsafe.Pointer) bool
	// gcdata stores the GC type data for the garbage collector.
	// If the KindGCProg bit is set in kind, gcdata is a GC program.
	// Otherwise it is a ptrmask bitmap. See mbitmap.go for details.
	gcdata    *byte
	str       nameOff
	ptrToThis typeOff
}

```


一个空 interface ：只有当  type data 都等于0时，才等于 nil

a := interface{}

a = nil

此时：
type = nil
data = nil



