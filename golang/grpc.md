# grpc

## 概要

Remote Procedure Protocol：远程调用协议，A机器上的代码，通过函数调用B机器上的函数。应用于分布式。

Google Remote Procedure Protocol ：谷歌出口的一个RPC框架

> 有趣的是这个东西居然有很多语言版本，它不是一个单独的软件，可以把包嵌入到项目中使用，这个跟thfit差别好大。

支持如下语言：

C C\+\+ dart go java oc py ruby php node...

## 特点

1. 分布式
2. 跨语言
3. 统一的API定义文档

## 使用场景

1. 可以把复杂的大项拆分成若干个小的模块，再放到不同的服务器分布计算。
2. 项目的语言可以不局限一个，多语言 包容性 开发
3. 有一个统一的API接口文档，所有项目、语言统一维护

> 小项目，小公司没必要上这个东西，复杂度有点高。

## 安装

> 就以GO语言~为测试标准吧

先下2个包：

```
$ go install google.golang.org/protobuf/cmd/protoc-gen-go@v1.26
$ go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@v1.1
```

> protoc\-gen\-go 这东西我安protobuf的时候，已经安过了，跳过
> 
> 
> protoc\-gen\-go\-grpc 这个包好像不用安装，可能是因为安装过上面的包了吧

两个包都是辅助protoc脚本，生成特定语言的pb文件

这里注意：因为ETCD与高版本GRPC冲突，所以，只能降级protobuf grpc

protoc 3.5

protoc\-gen\-go@v1.2.0

## 使用

先看下目录结构：

```
├── make.sh
├── makepbservice.php   自动生成class-func的工具脚本
├── pb                 最终编译好，可用的GO文件
│   ├── common.pb.go    
│   └── demo.pb.go      
├── pbservice           自动生成func就是在这个文件夹子下
│   └── demo.proto.go   
└── proto
    ├── common.proto
    └── demo.proto     这个就是核心描述文件 
```

demo.proto,如下：

```
syntax = "proto3";

import "proto/common.proto";

package pb;
option go_package ="./;pb";

service First {
  rpc SayHello (RequestRegPlayer) returns (ResponseReg) {}
  rpc SayHi (RequestRegPlayer) returns (ResponseReg) {}
}

message RequestRegPlayer{
    int32               add_time    = 1;
    repeated    Player              player_list = 3;
    Player              player_info = 4;
    map<fixed64,Channel> channel_map = 5;
}

message Player{
    uint64       id             = 1;
    bytes       role_name       = 2;
    string      nickname        = 3;
    fixed32     status          = 4;
    float       score           = 5;
    PhoneType   phone_type      = 6;
    bool        sex            = 7;
    sint32      level           = 8;
}

message Channel{
    sint64 id = 1;
    string name = 2;
}

enum PhoneType {
    //allow_alias = true;
    mobile = 0;
    home = 1;
    word = 2;
}

message ResponseReg{
    bool    rs  = 1;
}

message MyHeader {
    Common common = 1;
}
```

这里定义了一个服务叫：First

## 生成PB文件

网上的方法：

> protoc \-I helloworld/ helloworld/helloworld.proto \-\-go\_out=plugins=grpc:helloworld

高版本 protoc 方法

> protoc \-\-go\-grpc\_out=./pb ./proto/demo.proto

旧版本 protoc 方法\(因为我的ETCD只能用grpc旧版，protoc也得跟着降版本\)

> protoc \-\-go\_out=plugins=grpc:. ./proto/demo.proto

## 代码阶段

server 和 client

详细的代码就不列了，自己百度吧

其中核心：监听一个TCP的IP:PORT，把这个监听传给上面生成好的PB即可。

## 原理

内部使用的是：http2.0，也就是说，虽然我开的是 TCP 协议监听，但整个交互是在TCP层面上又以HTTP传输。

HTTP2.0优缺点请跳转~

## grpc-http2-协议

既然是HTTP协议，那大体上相同：

HTTP头\+HTTP体

HTTP头=path\(/包名.服务名/方法名\)\+content\-type\(application/grpc\)

HTTP体=grpc包头\+业务数据

grpc包头=type类型位\+压缩标识位\+数据长度

数据读出来，根据方法名再找对应的proto类，写到类中，传给监听回调函数即完成

不太复杂感觉

## 实际应用

demo的方式，有些问题需要思考：

1. 每编译还得手动创建一个结构体，再挂上相关方法，有点麻烦
2. 如果有透传参数咋？
3. 如果有些公共处事件理咋办？如：日志、metrics等
4. 假设第1条生效，那能不能把用户的handle加进去呢？
5. 假设第4条成立，那么统一返回的值，是不是还得再封装一下？

解决办法：

1. 写个脚本，去分析.proto文件，帮忙生成相关类名\+函数名
2. 使用 metadata 透传值，毕竟是HTTP2.0，肯定有header
3. 第1条里的，加入公共代码
4. \(1\) 可以自己写，但牵涉到动态路由，有点麻烦呢。
    \(2\) c/s端增加拦截器
5. \(1\) 可以自己写
    \(2\) c/s端增加拦截器

## 拦截器

1. UnaryInterceptor 一元拦截器
2. StreamInterceptor 流拦截器
3. clientInterceptor 客户端

## rpc 对比 restful

1. rpc 性能要略好一些，毕竟是二进制传输，但依然还是HTTP协议。
2. rpc 可能更倾向于内部使用，分布式配合计算
3. restful 更适合一些对外的接口。
4. restful更简单，但凡有个浏览器都能访问。
5. rpc 倾向C/S结构，restful货币B/S结构

个人还是倾向restful，因为简单好用，调试容易~

## 总结

高级应用就得考虑各种各样的问题，而一旦考虑的多，就得聚合很多的代码进去，如：log metrics 链路追踪 错误报警 动态路由 统一回调结构体等等，又把项目复杂化了。原本如使用restful就挺简单，现在要加protobuf grpc库\(可能版本有兼容问题\)，再得加上编译过程\(学习protoœ\)，还得分配好目录结构，最后再加上这一堆代码...天呐~

rpc 只是个协议，GRPC thrif 才是真正实现了该协议的软件。但单看GRPC使用的HTTP2也没啥特别高深的东西，或者说没有实质性的创新与冲破。所以，没必要神话RPC，只是一种工具，具体还是得看使用场景。

## php、 go、protobuf

主要通信的就这3个东西，PHP \-\> protobuf \-\> GO

go安装比较简单，麻烦的是，安装 grpc 这个库，国内把google墙了，就得通过github 中转

先安装protobuf

wget protobuf\-all\-3.9.0.tar

tar \-zxvf

./autogen.sh

./configure \-\-prefix=/soft/protobuf

make

cp protoc /usr/bin/

protoc \-\-version

PHP扩展

wget https://pecl.php.net/get/grpc\-1.22.0.tgz

phpize

wget https://pecl.php.net/get/protobuf\-3.9.0.tgz

phpize

修改php.ini

extension=grpc.so

extension=protobuf.so

php \-m

这个扩展，对 GCC 有版本要求 。4.5 以上，所以还得再升级下GCC

wget http://gcc.skazkaforyou.com/releases/gcc\-4.8.2/gcc\-4.8.2.tar.gz

gcc \-v, g\+\+ \-v

这个是官方的grpc项目

git clone \-b $\(curl \-L https://grpc.io/release\) https://github.com/grpc/grpc

git clone https://github.com/grpc/grpc.git

//以上两个都行

//获取该项目的依赖的其它的包

git pull \-\-recurse\-submodules && git submodule update \-\-init \-\-recursive

//编译安装

make

make install

bin 下面的可执行文件，PHP在protoc.exe 文件的时候，需要个工具

cd examples/php

//安装 composer

curl \-sS https://getcomposer.org/installer | php

//用composer 安装 protobuf grpc ，工具类

php composer.phar install

//安装过程中，可能会报错，改下源

composer config repo.packagist composer https://packagist.phpcomposer.com

安装完后，会多一个vendor，查看下 protobuf grpc 文件都在不在

===========================================================

https://blog.csdn.net/java060515/article/details/84940938

wget https://dl.google.com/go/go1.12.7.linux\-amd64.tar.gz

解压

mv /soft/go

export GOROOT=/soft/go

export PATH="_P__A__T__H_:GOPATH/bin"

export GOWORK=/data0/www/gowork

export GOPATH=/data0/www/gowork //go get 目录

go get \-u github.com/golang/protobuf/proto

go get \-a github.com/golang/protobuf/protoc\-gen\-go

//很大概率会被墙，于是换套路

go install google.golang.org/grpc

src下新建 google.golang.org

cd google.golang.org

wget https://github.com/grpc/grpc\-go/archive/master.tar.gz

tar \-zxvf

mv grpc\-go\-master/ grpc

go install google.golang.org/grpc

依然会报错，大致是缺少依赖包，接着继续 安装

mkdir \-p src/golang.com/x

\#\!/bin/bash

MODULES="crypto net oauth2 sys text tools"

for module in _M__O__D__U__L__E__S__d__o__w__g__e__t__h__t__t__p__s_://_g__i__t__h__u__b_._c__o__m_/_g__o__l__a__n__g_/{module}/archive/master.tar.gz \-O _G__O__P__A__T__H_/_s__r__c_/_g__o__l__a__n__g_._o__r__g_/_x_/{module}.tar.gz

cd ${GOPATH}/src/golang.org/x && tar zxvf ${module}.tar.gz && mv ${module}\-master/ ${module}

done

依然报错，继续安装

wget https://github.com/google/go\-genproto/archive/master.tar.gz \-O ${GOPATH}/src/google.golang.org/genproto.tar.gz

cd ${GOPATH}/src/google.golang.org && tar zxvf genproto.tar.gz && mv go\-genproto\-master genproto

安装完成~NND
