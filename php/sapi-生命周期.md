
# 概览

Server Application Programming Interface：服务端应用编程端口



# PHP的执行方式

|          |        |     |
| :------- | :----- | :-- |
| cli 指令行  | shell  |     |
| 模块化/嵌入式  | apache |     |
| fast-cgi | nginx  |     |

正常一门语言，指令行模式是最普遍使用的。
但 PHP 还可以嵌入到某个软件内部执行，还可以外部套一个进程，实现某种协力，再交互。
总之：PHP 的执行方式很灵活，其核心是ZEND,在ZEND 上层再套一个SAPI，可以兼容更多的运行模式，使用率更广。


apache：PHP会被编译出一个.so 类库，嵌入到 apache 里的。用于当做 PHP 与 apache 的桥梁进行通信。
nginx：两边实现了 fastCgi 协议。PHP 增加了 fpm 模式，启动后，接管 nginx 的请求。


# 生命周期

PHP启动/一次请求的步骤如下：

| 执行步骤          | 描述                        |
| :------------ | :------------------------ |
| PHP_MINIT     | PHP启动时，每个模块初始化阶段，只执行一次    |
| PHP_RINIT     | 每次有请求过来时，会执行              |
| PHP_RSHUTDOWN | 每次有请求结束时，会执行              |
| PHP_MSHUTDOWN | PHP关闭模块 ，每个模块进行收尾工作，只执行一次 |

MINIT：APACHE在第一启动后，会再关闭模块，用于检查模块的配置文件，这个阶段模块可以自定义一些宏变量，同时会常驻内存

初始化若干全局变量、初始化若干常量、始化Zend引擎和核心组件、解析php.ini、全局操作函数的初始化、初始化静态构建的模块和共享模块\(MINIT\)、禁用函数和类、ACTIVATION、激活SAPI、环境初始化、模块请求初始化



# sapi

当PHP 被嵌入到 APACHE 中后，其中它就是一个完整的模块，像是一个图灵机：只有一个输入  和  输出。具体输入的参数 和 输出的参数规则，就是 sapi 

有了 sapi ，间接等于PHP更加通用化，可以适用很多场景，如：APACHE NGINX