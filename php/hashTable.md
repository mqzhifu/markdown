![[zend_hash_table.png]]

# 结构体

#### \_hashtable

```c
typedef struct _hashtable { 
    uint nTableSize;        // hash Bucket的大小，最小为8，以 2x 增长。
    uint nTableMask;        // nTableSize-1， 掩码，用于根据hash值计算存储位置。
    uint nNumOfElements;    // hash Bucket中当前存在的元素个数，count()函数会直接返回此值 
    ulong nNextFreeElement; // 下一个数字索引的位置，$arr[] = "hello"时会用到
    Bucket *pInternalPointer;   // 当前遍历的指针（foreach比for快的原因之一）
    Bucket *pListHead;          // 存储数组头元素指针
    Bucket *pListTail;          // 存储数组尾元素指针
    Bucket **arBuckets;         // 存储hash数组
    dtor_func_t pDestructor;  //在删除元素时执行的回调函数，用于资源的释放。
    zend_bool persistent; //指出了Bucket内存分配的方式。如果persisient为TRUE，则使用操作系统本身的内存分配函数为Bucket分配内存，否则使用PHP的内存分配函数
    unsigned char nApplyCount; // 标记当前hash Bucket被递归访问的次数（防止多次递归）
    zend_bool bApplyProtection;// 标记当前hash桶允许不允许多次访问，不允许时，最多只能递归3次
#if ZEND_DEBUG
    int inconsistent;
#endif
} HashTable;
```

| key              | value                            |                   |     |
| :--------------- | :------------------------------- | :---------------- | --- |
| nTableSize       | hash Bucket的大小，最小为8，以 2x 增长      |                   |     |
| nTableMask       | nTableSize-1， 掩码，用于根据hash值计算存储位置 |                   |     |
| nNumOfElements   | hash Bucket 中当前存在的元素个数           | count 函数使用        |     |
| nNextFreeElement | 指向下一个空的元素位置                      | foreach比for快的原因之一 |     |
| pInternalPointer | 当前遍历的指针                          |                   |     |
| pListHead        | 存储数组头元素指针                        |                   |     |
| pListTail        | 存储数组尾元素指针                        |                   |     |
| arBuckets        | 存储 hash 数组                       | 数组-保证线性顺序         |     |
|                  |                                  |                   |     |
#### bucket

```c
typedef struct bucket {
    /* Used for numeric indexing */
    ulong h;            // 对char *key进行hash后的值，数字索引的话就是索引值
    uint nKeyLength;    // hash关键字的长度，如果数组索引为数字，此值为0
    void *pData;        // 指向value，一般是用户数据的副本，如果是指针数据，则指向pDataPtr
    void *pDataPtr;     //如果是指针数据，此值会指向真正的value，同时上面pData会指向此值
    struct bucket *pListNext;   // 整个hash表的下一元素
    struct bucket *pListLast;   // 整个哈希表该元素的上一个元素
    struct bucket *pNext;       // 存放在同一个hash Bucket内的下一个元素
    struct bucket *pLast;       // 同一个哈希bucket的上一个元素
    char arKey[1];  
    /*存储字符索引，此项必须放在最未尾，因为此处只字义了1个字节，存储的实际上是指向char *key的值，
    这就意味着可以省去再赋值一次的消耗，而且，有时此值并不需要，所以同时还节省了空间。
    */
} Bucket;
```

|            |                                        |     |
| :--------- | :------------------------------------- | :-- |
| h          | hash 值，下标为数字索引时，h就是索引值                 |     |
| nKeyLength | key字符串的长度，当nKeyLength为0时表示是数字索引        |     |
| pData      | 指向value，一般是用户数据的副本，如果是指针数据，则指向pDataPtr |     |
| pDataPtr   | 如果是指针数据，此值会指向真正的value，同时上面pData会指向此值   |     |
| pListNext  |                                        |     |
| pListLast  | Hash拉链的上一个元素                           |     |
| pNext      | Hash拉链的下一个元素                           |     |
| pLast      |                                        |     |
| arKey      | key字符串指针                               |     |
|            |                                        |     |

# hash 函数

```c
typedef struct _zend_hash_key {
	char *arKey;      /* hash元素key名称 */
	uint nKeyLength;     /* hash 元素key长度 */
	ulong h;       /* key计算出的hash值或直接指定的数值下标 */
} zend_hash_key;
```


```c
h = zend_inline_hash_func(arKey, nKeyLength);   /* 计算key的hash值 */
nIndex = h & ht->nTableMask;    /* 利用掩码得到key的实际存储位置 */
```

>hash 函数：time33 ( key , 数组长度 ) = h

# 数字下标与字符串KEY共存


hash func(number|string) = 一个 uint 整数

不管是 int 还是 string 做为 key ，最终都能转换成一个整数。
但整数范围很大，数组下标却是有上限的，int太大，所以再经过一个掩网运算，就可以 得到 数组的下标，做为KEY了

bucket中：
- nKeyLength > 0 ，证明是字符串的KEY，
- nKeyLength = 0 ，即是数字索引。

# 扩容 

哈希表可用元素过半 且 h 在 两倍 nTableSize 范围内 则进行扩容
>nNumUsed >= nTableSize

策略：
- 数据量并没有那么大，删除元素
- 直接2N 扩容 

步骤：
1. 申请2N内存空间
2. 重新计算旧数据的 hash 值
3. 把旧的放到新的容器中
4. 释放旧容器
# 总结：

 hashtable ->buckers list array->bucket

hashtable 控制一个数组，数组中存储若干bucket
bucket 内部有两个双向链表：
- 指向上下的 bucket 元素
- 当发生冲突，指向左右的 bucket 元素

相对挺简单的，就是他的结构体里有很多成员变量，像：两组链表上下指针。
但，够用，PHP本就一次执行释放所有内容



