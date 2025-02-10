# alfred

这个真你妈的是神器，就是一个搜索框，往里可以输入任何内容，还包括指令

不过有个硬伤：他是依赖mac自带的spotlight，而spotlight是依赖mds做索引，而mds这个吃CPU

## 初始化

1. 打开后，先申请下：系统权限
2. 设置下快捷键
3. 设置搜索目录
4. 添加百度web搜索条目

## 主要功能

1. 文件/目录 搜索
2. 快捷启动软件
3. 快速启动浏览器并搜索
4. 插件，嵌入各种插件方便日常使用，像 有道翻译、 UNIXSTAME时间转换、md5 等
5. 快速执行系统指令，如：shutdown restart gc quitAll等
6. 计算器

具体：

> 1、自定义搜索的目录，搜索任何东西
> 
> 
> 2、open 开头，可以搜索\+打开文件
> 
> 
> 3、find 开头，可以搜索 文件具体位置，在FIND中打开
> 
> 
> 4、baidu开头，可直接在默认浏览器中搜索 关键词
> 
> 
> 5、计算机，可直接在搜索框内 输入数字\+运算符号
> 
> 
> 6、define 开头，加任意单词，可翻译
> 
> 
> 7、spell 开头，想不起哪个单词，可帮助提示补全

## 日常 workflows 列表

|指令   |说明                              |
|-------|----------------------------------|
|yd     |快捷使用有道词典翻译              |
|hash   |md5 sha 加密串                    |
|time   |unix时间转换                      |
|github |快速在gt上查找项目或找开自己的项目|
|encode |url encode/decode 转换器          |
|NSC    |各种进制转换                      |
|codeVar|给变量起名                        |

## 日常OS操作指令

先：修改清空垃圾桶的快捷键为gc

|指令    |说明                  |
|--------|----------------------|
|gc      |清空回收站            |
|restart |重启                  |
|shutdown|关机                  |
|quitall |关闭当前所有已打开软件|
|about   |关于本机              |

设置搜索目录

```
/Applications
/Applications/Xcode.app/Contents/Applications
/Library/PreferencePanes
/System/Library/PreferencePanes
/usr/local/Cellar

/System/Applications
/data/www

```

## 有道

去有道云智能中，找配置信息

https://ai.youdao.com/\#/

手机号登陆

创建应用 文本翻译 勾选 接入方式：api

业务总览：

appid：

5b257a9c33bd6d51

应用密钥

1Yc6rq2LXuUSw83mMz2pEo8uorH6peN3

下载workflow:

https://github.com/wensonsmith/YoudaoTranslator/releases/download/3.1.0/YoudaoTranslator\-3.1.zip

拖入alfred中

点击右上角的 X 图标，选择 选项卡：环境变量，把appid 密钥 设置进去

## hash

https://github.com/willfarrell/alfred\-hash\-workflow

## TimeStamp

https://github.com/WiconWang/Alfred\-Workflows\-TimeStamp

time 2017\-03\-07 20:14:48

time 1488888888

time now

## github

https://github.com/gharlan/alfred\-github\-workflow/releases

## 创建文件

https://github.com/SteliosHa/Alfred\_File\_Creator
