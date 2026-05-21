
# Pool

内存对象池。(不是内存池，是已帮助实例化好的对象的内存池)


我们日常业务，经常有：开一个 1000 大小的字节 的 buffer ，一次使用即不会再用了，等GC回收。

如果，有个池子：里面存在着大量的这种1000大小的BUFFER，用完也不释放，GC会自动释放。 

第一次使用后会设置某些值，第二次再拿出来是带着旧值的，你得重置一下。

每个处理器（P）都分配了 本地私有池（poolLocal）。


# 私有对象池怎么设计？

对比传统POOL:
- 更可控:有多少对象 什么时候被回收
- 不会被GC清
- 能限制数量
- 能做分级:1KB 10KB
- 能做无锁
- 能做内存管理



freelist（空闲链表）


# 内存对齐


CPU如果是64位的，其实是地址/数据总线是64位的宽度。也就是：一次最小内存的IO，能获取数据:8 x 8个字节 = 64二进制位。

假设：当前数据存储3个数据：  64+1+8，此时要获取 64和1
65个位，得两次内存IO，且CPU还得拼接：  64+1，不然1后面其实还有数据，得清空掉

换个思路，64+1+63个空位+8，此时：
两次内存IO，就是调用方想要的数据了，不用再拼接了





```
type T struct {  
	a bool  
	b int64  
	c bool  
}
```


你以为：1 + 8 + 1 = 10
实际 ：unsafe.Sizeof(T{}) = 24

8字节对齐：
```
a 1 byte  
padding 7 byte  
b 8 byte  
c 1 byte  
padding 7 byte
```

优化：
```
type T struct {  
	b int64  
	a bool  
	c bool  
}
```



把占用空间大的，放到最上方，最小的放到最下方



# 零拷贝 string/[]byte？

string
```
type stringStruct struct {  
	data *byte  
	len int  
}
```


string 是readonly的，不可变的


[]byte：
```
type slice struct {
    data *byte
    len  int
    cap  int
}
```

这是个切片。

a := []byte("hello")
b := string(a)
GO，此时会 复制一份内存

```
b := unsafe.Slice(unsafe.StringData(s), len(s))
```


a := "hello"
b := []byte(a)

GO，此时也会 复制一份内存


```
s := unsafe.String(&b[0], len(b))
```


尽量少用：string 拼接 string，或者 string()转换，因为会copy出新的内存
```
for i := 0; i < n; i++ {  
	s += "a"  
}
```

s= "a",s="aa",s="aaa",s="aaaa....."  ，一次就产生了N个这样的对象

优化使用：Builder、strings.Join



