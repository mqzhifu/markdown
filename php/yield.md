
# 概览

当一个函数中有 yield 出现时，调用后，会返回一个 迭代器，函数原来的执行流程发生了变化
>函数执行时：遇到 yield 后，程序会立刻停止，并返回一个值，后面的代码不会再执行



# 例子

```php
function PrintInfo()  
{  
	for ($i = 0; $i < 3; $i++) {  
		echo "输出存在感1\n";  
		yield 1;  
		echo "输出存在感2\n";  
	}  
}  
  
$i = 0;  
$g = PrintInfo();  
var_dump($g);  
  
foreach ($g as $value) {  
	$i++;  
	echo $i.":".$value."\n";  
}

```

PrintInfo 函数，循环3次，输出一些值

调用：PrintInfo 函数 ，它会返回一个迭代器
我们遍历这个迭代器：foreach

循环中，第一次执行   到 yield 停止执行，并返回 1
循环中，第二次执行   到 yield 停止执行，并返回 1
循环中，第三次执行   到 yield 停止执行，并返回 1


输出结果，如下：
```
object(Generator)#1 (0) {
}
输出存在感1
1:1
输出存在感2
输出存在感1
2:1
输出存在感2
输出存在感1
3:1
输出存在感2
```


换一种实现方式：
```php
function readTxt(){
	$handle = fopen("./test.txt", 'rb');
	while (feof($handle)===false) {
		yield fgets($handle);
	}

	fclose($handle);

}

foreach (readTxt() as $key => $value) {
	echo $value.'<br />';

}
```

这就是官方给的一个 DEMO，大致意思：读取一个文件内容，与传统写法的区别是：
要有一个数组，把文件的内存依次全读出来，最后放到数组中。
这样做，节省了一个数组，也算节省了空间。
>感觉没太大差别，传统的写法，不用数组保存，一行一行的做处理，不也一样嘛？

既然是迭代器，也不一定非得用 foreach ，还有一些函数

# 函数

```php
$gen->current();//执行当前 代码
$gen->next();//继续执行 后面代码
$gen->valid();//判断是否已执行到最后一个元素了
$gen->send();//给yield 函数 内部发送参数
```

# 协程调度器

```php
function aaa(){

	yield init()
	
	yield process()
	
	yield end()

}

function bbb(){

	yield init()
	
	yield process()
	
	yield end()
}

```

通过 yield 就实现了，可以先执行 aaa 中的 init 方法，接着执行 bbb 中的 init() 方法

# 总结

很鸡肋。如果只会PHP一门语言，有点用。但如果会C/C++ JAVA PY C# ，这东西真的是一点没有。
主要： yield 并不是并行计算，它还是单进程/线程 顺序执行

