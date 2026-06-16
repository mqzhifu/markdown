
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
## 项目目录-结构

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


# mod 包管理


| Go 版本 | 发布时间       | 状态        | 说明                                       |
| ----- | ---------- | --------- | ---------------------------------------- |
| 1.11  | 2018 年 8 月 | **实验性引入** | 可以用，但需要开启环境变量 `GO111MODULE=on`           |
| 1.13  | 2019 年 9 月 | **默认开启**  | **正式成为标准**，不需要手动开启，大部分公司开始切换             |
| 1.16  | 2021 年 2 月 | **完全稳定**  | `go install`/`go get` 分工明确，**现在你用的就是这套** |

强依赖：$HOME 环境变量。
当有新包下载后，会在 $HOME 目录下创建一个叫:go 的文件夹，且包含目录：pkg bin

bin：go install 安装的可执行文件
pkg
- mod/ # 第三方库源码+版本缓存（最重要） 
 - darwin_amd64/ 编译缓存（.a 文件）


| 命令              | 核心用途                   | 是否修改 go.mod | 是否编译二进制     | 是否安装到 `$HOME/go/bin` | 是否仅下载源码缓存   | 日常开发使用场景              |
| --------------- | ---------------------- | ----------- | ----------- | -------------------- | ----------- | --------------------- |
| **go get**      | 管理项目依赖、新增 / 升级 / 降级库版本 | ✅ 会修改       | ❌ 不编译程序     | ❌ 不安装                | ⚠️ 顺带下载到缓存  | 手动指定依赖版本、升级降级包        |
| **go install**  | 编译源码并安装**全局 CLI 工具**   | ❌ 不修改       | ✅ 编译生成可执行文件 | ✅ 安装到全局 bin 目录       | ❌ 不只下载，还要编译 | 安装热重载、代码格式化、LSP 等全局工具 |
| **go download** | 仅下载模块源码到本地缓存，无额外操作     | ❌ 不修改       | ❌ 不编译       | ❌ 不安装                | ✅ 只下载缓存     | 离线环境预拉依赖、提前缓存源码，几乎很少用 |

go get 会升级包的版本，如：带 @latest


## 开启 mod 模式
>go env -w GO111MODULE=on

##  新建项目/初始化
>cd project_name
>go mod init

会在项目中生成 go.mod 和go.sum ,然后 vim go.mod

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

- module:包/项目名名称
- require:必须下载的包
- replace:将require的包替换成另外一个包

更新/下载/包(多余的包会被删除，没有的包会被下载)
> go mod tidy

将3方下载到当前目录的vendor 下

> go mod vender


GOSUMDB：（校验和数据库） 环境变量，用于验证下载的 Go Module 依赖包的完整性和真实性，防止依赖被恶意篡改或劫持。
 计算下载内容的哈希值（SHA256）
 向 GOSUMDB 服务器查询该模块版本的官方校验值
 比对本地哈希与服务器记录：
   ✓ 一致 → 写入 go.sum，继续构建
   ✗ 不一致 → 报错 "checksum mismatch"，终止构建


go env GOSUMDB
go env -w GOSUMDB=off

GO.SUM：go.mod里每引入一个包，go.sum就会遍历这个包的所有文件内容，最终生成一行记录，加到go.sum里



go get/go download/go install，下载包时，基本就是两个源：
- github
- golang官网
- 自己搭建的源

go tidy

|步骤|核心动作|具体细节|
|---|---|---|
|1. 扫描源码|递归遍历源码文件|遍历项目模块内所有目录，扫描全部 `.go` / `_test.go` 文件，提取代码中**所有 import 导入路径**|
|2. 解析依赖图|映射模块并拉取版本|将提取的 import 路径，映射为标准 `module@version`；本地无版本信息时，请求 `GOPROXY` 模块代理获取版本与元信息，构建完整依赖关系图|
|3. 增删 go.mod|补依赖、清冗余、标间接|1. 补充：代码已引用但 go.mod 未声明的依赖；<br><br>2. 清理：删除业务代码、测试代码均无引用的无用直接 / 间接依赖；<br><br>3. 标记：仅被第三方依赖引用、本项目未直接 import 的模块，自动添加 `// indirect` 注释|
|4. 版本选择|遵循 MVS 版本策略|按 **Minimal Version Selection 最小版本选择** 原则，选取能满足所有依赖约束的**最低兼容版本**，不盲目升级最新版，保证版本兼容稳定|
|5. 同步 go.sum|下载缓存 + 哈希校验维护|1. 本地 `pkg/mod` 无缓存时，自动下载对应模块到全局缓存；<br><br>2. 对模块完整 zip 包、模块内单独 `go.mod` 分别计算 **SHA256 哈希**，写入 go.sum；<br><br>3. 清理已废弃、被移除依赖的旧哈希记录，精简 go.sum|
|6. 排序格式化|规整文件顺序|对 go.mod 的 `require` 条目、go.sum 所有记录，按**字母字典序**自动排序格式化，保证多人协作、跨平台 Git Diff 稳定无乱序|
|7. 校验与报错|全链路合法性检查|校验网络连通性、模块版本冲突、go.sum 哈希校验和失败、私有模块 `GOPRIVATE` 配置异常等；校验失败立即中断流程，输出明确错误信息|



# 引地本地包

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
2. 此时，不需要 go get 了在，在执行 go run main.go后，会自动加载这些包

包的版本拉取规则：
1. 先拉取最新的release / tag，
2. 拉取最新的 master head commit

# IDE-配置

goland：jetbrains出品，直接去网上找吧

需要配置下：GO的路径、编译执行参数等

偏好设置->Go->Go moudles->environment中添加：

> GOPROXY=https://goproxy.cn;GO111MODULE=on

现在都是 MOD 模式了，比较简单，也用再配置：GOPATH GOROOT了



# 私有库搭建


```
export GOPRIVATE=git.mycompany.com
```

但，你的go.sum 将会失效。
https 也会失效，得配置 token 验证

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

创建临时目录
编译源代码
链接依赖项
生成临时程序
执行该程序
自动清理现场

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