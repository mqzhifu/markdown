
```php
echo "Hello World";
$a = 1 + 1;
echo $a;
```


PHP执行这段代码会经过如下4个步骤\(确切的来说，应该是PHP的语言引擎Zend\)

| keyword          | memo                          |     |
| :--------------- | :---------------------------- | --- |
| Scanning(Lexing) | 将PHP代码转换为语言片段(Tokens)         |     |
| Parsing          | 将 Tokens 转换成简单而有意义的表达式        |     |
| Compilation      | 将表达式编译成Opocdes                |     |
| Execution        | 顺次执行Opcodes，每次一条，从而实现PHP脚本的功能 |     |

>现在有的 Cache 比如 APC,可以使得PHP缓存住Opcodes，这样，每次有请求来临的时候，就不需要重复执行前面3步，从而能大幅的提高 PHP的执行速度。

PHP:解释型语言：词法分析-》语法分析-》opcode-》执行

# Lexing 词法分析

```php
Array
(
    [0] => Array
        (
           [0] => 367
           [1] =>  Array
        (
            [0] => 316
            [1] => echo
        )
    [2] => Array
        (
            [0] => 370
            [1] => 空格
        )
    [3] => Array
        (
            [0] => 315
            [1] => "Hello World"
        )
    [4] => ;
    [5] => Array
        (
            [0] => 370
            [1] =>
        )
    [6] => =
    [7] => Array
        (
            [0] => 370
            [1] =>
        )
    [8] => Array
        (
            [0] => 305
            [1] => 1
        )
    [9] => Array
        (
            [0] => 370
            [1] =>
        )
    [10] => +
    [11] => Array
        (
            [0] => 370
            [1] =>
        )
    [12] => Array
        (
            [0] => 305
            [1] => 1
        )
    [13] => ;
    [14] => Array
        (
            [0] => 370
            [1] =>
        )
    [15] => Array
        (
            [0] => 316
            [1] => echo
        )
    [16] => Array
        (
            [0] => 370
            [1] =>
        )
    [17] => ;
)
```


源码中的：空格、分号、tab都在。字符串、操作符、数字、语句 都出现了。
操作符，语句，都会被转换成一个包含俩部分的Array: Token ID
>Zend内部的改Token的对应码，比如,T_ECHO,T_STRING

# Parsing

首先会丢弃 Tokens Arra y中的多于的空格，然后将剩余的Tokens转换成一个一个的简单的表达式

1. echo a constant string
2. add two numbers together
3. store the result of the prior expression to a variable
4. echo a variable

# Compilation

它会把 Tokens 编译成一个个 op_array, 每个op_arrayd包含如下5个部分：

1. Opcode 数字的标识，指明了每个 op_array 的操作类型，比如add , echo
2. 结果 存放Opcode结果
3. 操作数1 给Opcode的操作数
4. 操作数2
5. 扩展值 1个整形用来区别被重载的操作符

比如，我们的PHP代码会被Parsing成:

```c
ZEND_ECHO  ‘Hello World’
ZEND\_ADD ~0 1 1
ZEND\_ASSIGN \!0 ~0
ZEND\_ECHO \!0
```


我们的$a去那里了？

恩，这个要介绍操作数了，每个操作数都是由以下俩个部分组成：

```c
1. a)op_type : 为 IS_CONST, IS_TMP_VAR, IS_VAR, IS_UNUSED, or IS_CV
2. b)u,一个联合体，根据 op_type 的不同，分别用不同的类型保存了这个操作数的值(const)或者左值(var)
```


而对于var来说，每个var也不一样

IS\_TMP\_VAR, 顾名思义，这个是一个临时变量，保存一些op\_array的结果，以便接下来的op\_array使用，这种的操作数的u保存着一个指向变量表的一个句柄（整数），这种操作数一般用~开头，比如~0,表示变量表的0号未知的临时变量

IS\_VAR 这种就是我们一般意义上的变量了,他们以$开头表示

IS\_CV 表示ZE2.1/PHP5.1以后的编译器使用的一种cache机制，这种变量保存着被它引用的变量的地址，当一个变量第一次被引用的时候，就会被CV起来，以后对这个变量的引用就不需要再次去查找active符号表了，CV变量以！开头表示。

这么看来，我们的$a被优化成!0了。

\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-

Zend Engine，调用词法分析器( Lex生成的，源文件在 Zend/zend\_language\_sanner.l\),
将我们要执行的PHP源文件，去掉空格 ，注释，分割成一个一个的token。

然后，ZE会将得到的 token forward 给语法分析器、(yacc生成, 源文件在 Zend/zend_language_parser.y)，生成一个一个的 op code，opcode一般会以op array的形式存在，它是 PHP 执行的中间语言。

最后，ZE调用zend\_executor来执行op array，输出结果。



ZE是一个虚拟机，CISC（复杂指令处理器）， 它支持150条指令（具体指令在 Zend/zend_vm_opcodes.h），包括从最简单的ZEND_ECHO(echo)到复杂的 ZEND_INCLUDE_OR_EVAL(include,require)，所有我们编写的PHP都会最终被处理为这150条指令(op code)的序列，从而最终被执行。

有什么办法可以看到我们的PHP脚本，最终被“翻译”成什么样的呢？ 
VLD\(Vulcan Logic Dissassembler)模块。
下载这个模块，并把他载入PHP中，就可以通过简单的设置，来得到脚本翻译的结果了。
具体关于这个模块的使用说 明

接下来，让我们尝试用VLD来查看一段简单的PHP脚本的中间语言。

原始代码：

\<?php

$i = “This is a string“;

//I am comments

echo $i.‘ that has been echoed to screen‘;

?\>

采用VLD得到的 op codes:

```c
filename:/home/Desktop/vldOutOne.php  
function name: (null)  
number of ops: 7  
line #  op                 fetch       ext  operands  
-------------------------------------------------------------------------------------------------------------------------------  
2 0 FETCH_W local $0, 'i'  
1 ASSIGN $0, 'This+is+a+string'  
4 2 FETCH_R local $2, 'i'  
3 CONCAT ~3, $2,'+that+has+been+echoed+to+screen'  
4 ECHO ~3  
6 5 RETURN 1  
6 ZEND_HANDLE_EXCEPTION
```

源文件中的注释，在 op code 数组中，已经没有了，不用担心注释影响你的脚本执行时间
>实际上，它是会影响ZE的词法处理阶段的用时而已

现在我们来一条一条的分析这段 op codes，每一条 op code 又叫做一条 op_line，都由如下7个部分，在zend_compile.h中，我们可以看到如下定义：

```c
struct _zend_op {  
	opcode_handler_t handler;  
	znode result;  
	znode op1;  
	znode op2;  
	ulong extended_value;  
	uint lineno;  
	zend_uchar opcode;  
};
```


其中，opcode 字段指明了这操作类型，handler 指明了处理器，然后有俩个操作数，和一个操作结果。
>_zend_op 这个结构就已经告诉 zend 如何执行一条 op_code 了

1. FETCH_W, 是以写的方式获取一个变量，此处是获取变量名”i”的变量于$0（*zval）。
2. 将字符串”this+is+a+string”赋值(ASSIGN)给$0
3. 字符串连接
4. 显示

真正负责执行的函数是，zend\_execute, 查看zend_execute.h:

>1. ZEND_API extern void (*zend_execute)(zend_op_array *op_array TSRMLS_DC);

 \_zend_op_array 结构体略长，我就不写了，我大概看了一下里面的值：感觉像是一个函数的上下文，里面还有  scope 成员，参数等等，上面的的 op_code 在这个 执行环境中 就可以完整的执行了



