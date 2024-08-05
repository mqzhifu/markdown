
# 官网 

GO下载地址：
>https://golang.google.cn/dl/

各种包下载：
>https://pkg.go.dev/


查看版本
> go version

查看 GO 配置环境
> go env


# 包管理/项目结构（旧）

go 1.11 之前的版本，均使用go get 下载-包 ，且依赖 环境变量：GOPATH，下载的包会在：GOPATH/pkg 下面。如果是下载+安装可执行文件，会在 GOPATH/bin  目录下

获取一个包：

> go get http://xxxxx/packageName@版本号

更新一个包

> go get -u xxxxx/package


#### 环境目录/变量

|        |                        |     |     |
| :----- | :--------------------- | :-- | --- |
| GOROOT | golang 语言安装目录          |     |     |
| GOPATH | 指向项目目录，其包含：src pkg bin |     |     |
|        |                        |     |     |

#### 项目目录-结构


| 目录名 | 解释     |     |     |
| :-- | :----- | :-- | --- |
| src | 源码     |     |     |
| pkg | 下载的3方包 |     |     |
| bin | 可执行的工具 |     |     |

缺点:

1. 该方法是强依赖 $GOPAH 设置的路径
2. 无法做到:每个项目->使用相同的包->但是版本不同
3. 依赖src pkg bin 3个文件夹

简单理解:无法做到包的版本管理，没有像 PHP 中 composer 的配置文件

宏观看：

我一个项目非要搞它3个文件夹子确实有点扯~另外，还得再配置一个LINUX环境变量，虽然JAVA也这么干，但是毕竟是PHP转过来的，能省一步就是一步啊


# 包管理/项目结构（新）


go 1.11 之后的版本，可使用 mod 方式，并不强行依赖 gopath。而是依赖：$HOME 环境变量。
当有新包下载后，会在 $HOME 目录下创建一个叫:go 的文件夹，且包含目录：pkg bin

感觉跟 php composer 差不多，在项目的根目录加一个配置文件，可下载到公共目录 也可以下载到当前项目中 vendor


#### 开启mod模式
>go env -w GO111MODULE=on

GO111MODULE 有3个值： on auto off，默认是 off

#### 空项目-使用 初始化
>go mod init

会在项目中生成go.mod 和go.sum ,然后 vim go.mod

```
module frame_sync

go 1.14

require (
	github.com/golang/protobuf v1.5.0
	github.com/gorilla/websocket v1.4.2
	google.golang.org/protobuf v1.26.0
	zlib v0.0.0
)

replace zlib v0.0.0 => ../zlib

```

- module:包名称
- require:必须下载的包
- replace:将require的包替换成另外一个包

更新/下载/包(多余的包会被删除，没有的包会被下载)
> go mod tidy

将3方下载到当前目录的vendor 下

> go mod vender

吐槽:感觉跟 php composer 一模一样，并且PHP的功能更强....


下载一个包，完成后，进行编译，生成可执行文件，并放入bin目录：
>go install xxxxxxx

编译项目，最后将编译后的过程文件，会添加到pkg下，同时在bin/下再添加一个可执行文件

# 包管理-总结


旧的方法：更松散一些，可能早期也是够用，没在在意规范这事儿
MOD： 更倾向于现代式的模式。类似于：composer meven 模式，可基于某一个项目

GOLANG的包基本就是两个源：
- github
- golang官网

冲突点

1. 一但包里有go.mod，那么再执行go get 将被mod 控制
2. mod也使用gopath ，用于存储下载包的位置

两者差别

1. imports包路径不一样了
2. 本地包引入，得改成 间接引入
3. 旧下包是在src下，mod是gopath/pkg/mod下
4. 旧的go get 是循环下包，包括子包，mod不支持子包下载，只是单纯的 download
5. 旧方式 go get 是git clone \+ go install ,mod 只是go clone

引地本地包

```
require (
    test v0.0.0
)

replace (
 test => ../test
)
```

# 自己制作-包

官网找不到的包，也会自动去 github 上找：
1. 去github上的项目中，随便挑个项目，先打一个v0.0.1 的版本,release
2. 此时，不需要go get 了在，在执行 go run main.go后，会自动加载这些包

包的版本拉取规则：
1. 先拉取最新的release / tag，
2. 拉取最新的 master head commit

# IDE-配置

goland：jetbrains出品，直接去网上找吧

需要配置下：GO的路径、编译执行参数等

偏好设置->Go->Go moudles->environment中添加：

> GOPROXY=https://goproxy.cn;GO111MODULE=on

现在都是 MOD 模式了，比较简单，也用再配置：GOPATH GOROOT了

# GO-编译过程

go build：编译项目代码
如果不指针入口文件，默认是：main.go，会生成一个临时文件(如没指定文件名，即：main
如果没有设置 path，即在当前文件下生成此文件

go run：编译并运行(包含main.go并可执行)，不会生成临时文件

go bulid -x :编译，同时可以查看到编译了哪些文件

手动分拆编译：
>go tool compile hello.go go tool link hello.o

编译过程 ：文本代码->目标文件 .o .a ->连接 ->可执行文件

如果有使用 CGO ，需要添加其它参数

# CGO

有时候GO里想直接调用C/C++的一些公共函数或类库，像：dll so 文件。
类库，如：ffmpeg ，处理音视频流等

>因为GO是C和汇编写的，且也是静态语言，调用C/C++相对容易些

```golang
/*
#include <stdio.h>
#include <stdlib.h>
*/

import "C"
C.print(a)


```

前置条件：GCC
如果是自己写的C/C++代码，得用GCC编译
如果是3方已编译好的:dll so 动态链接库，只需要注意一下OS及GCC


编译：
CGO_ENABLED : 1=开启CGO 0=关闭CGO。默认为1
GOOS:平台系统
GOARCH:平台架构（32位、64位）
-ldflags：一些参数，如：静态编译


C/C++ 里也可以反调GO，但我没用过，也没研究过，不做讲解

缺点：
- OS兼容问题，这里尤其是WIN下就没有GCC，即使有生成的DLL也是个问题，MAC也差不多
- GCC 版本，决定了编译出来的库，哪些 OS 上能用，也算是OS兼容问题
- 编译麻烦，得加很多参数，且编译时间加长
- 维护麻烦，还得再维护一套C/C++代码，且有时使用的是编译后的库，非明文代码，连查看都是个问题
- 如果是信赖3方较大的库，3方库有变化，项目代码也得跟着变
- 对C/C++不熟悉 ，或代码写的不够好，像内存溢出、GC 等等，问题很多的

我是真心不推荐使用这东西，日常的项目都写不好，还搞C/C++，除非是像使用 ffmpeg 这种没办法的库。否则，只会增加复杂度，毫无意义 。