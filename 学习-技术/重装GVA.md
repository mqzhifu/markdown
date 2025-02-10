# 重装GVA

## gin\-admin\-vue

一个可视化管理台后\(web 浏览器 BS结构\)，带一些基础功能。简化重复造轮子。

主体是：GO\+VUE

实现了：前后端分离、MVC

官网

> https://www.gin\-vue\-admin.com/docs/first\_master

下代码：

> git clone https://github.com/flipped\-aurora/gin\-vue\-admin.git

代码结构主要分为两个部分：server 和 web

## 下载

官方地址：

> git clone https://github.com/flipped\-aurora/gin\-vue\-admin.git

自己项目的地址:

> git http://192.168.1.22:40080/frontend/seed\_admin\_vue.git
> 
> 
> git http://192.168.1.22:40080/backend/seed\_admin\_go.git

下载完成后，修改下权限：

> chown \-R wangdongyan:\_www /data/www/golang/seed\_admin\_vue
> 
> 
> chown \-R wangdongyan:\_www /data/www/golang/seed\_admin\_go

把cicd脚本复制进去：

> cp cicd.toml /data/www/golang/seed\_admin\_vue
> 
> 
> cp cicd.toml /data/www/golang/seed\_admin\_go

## 安装前端

cd seed\_admin\_vue

> npm install \-\-unsafe\-perm=true \-\-allow\-root

安装不成功的话，换个淘宝的源：

> npm install \-g cnpm \-\-registry=https://registry.npm.taobao.org
> 
> 
> cnpm install \-\-unsafe\-perm=true \-\-allow\-root

两种方式均可，官方更推荐npm，但速度没有cnpm快

启动:

> npm server

查看下端口号：

lsof \-i:8080

配置文件修改：

```
vim .env.product
>VITE_BASE_API = http://adminapi.seedreality.com
```

vim vite.config.js

> outDir: '../dist'

产出目录,修改编译JS输出目录

## 安装后端

修改 mod 的包名，也可以不改，它这个就是有点长...

> github.com/flipped\-aurora/gin\-vue\-admin/server \> Zwebuigo

mod下包，并启动后端:

```
cd seed_admin_go
go mod tidy
go run main
```

修改一下自动生成代码的目录:

vim config.yaml

```
autocode->server->/seed_admin_go
autocode->web->/seed_admin_vue/src
```

## 初始化DB

> http://127.0.0.1:8080

点击：初始化按钮，输入MYSQL配置信息，执行即可

## 页面修改

#### 删除默认密码/始化按钮

view/login/login.vue

168行：

删除admin 和 123456

#### 换LOGO

#### 删除尾部

#### 个人信息，修改头部 手机号 地址 邮箱 等等

## DIY\-前端代码操作

#### 添加2个自定义常用的函数

vim web/src/utils/format.js

```
export const formatUnixDate = (time) => {
  if (time !== null && time !== '' && time > 0) {
    time = time + "000"
    time = parseInt(time);
    var date = new Date(time)
    return formatTimeToStr(date, 'yyyy-MM-dd hh:mm:ss')
  } else {
    return '--'
  }
}
```

date字段标识添加:formatter="substrContent"

```
const substrContent = (val) => {
  var v = val.headerCommon
  if( v.length > 40){
    return v.substr(0,40)
  }else{
    return v
  }
}
```

## DIY\-后端代码

#### 修改配置文件，增加一个DB配置项：

vim config.yaml

```
mysql-business:
  path: 8.142.177.235
  port: "3306"
  config: charset=utf8mb4&parseTime=True&loc=Local
  db-name: "seed_pre"
  username: root
  password: ''
  max-idle-conns: 10
  max-open-conns: 100
  log-mode: error
  log-zap: false
```

给DB增加公共配置，DB选项

vim config/config.go

```
MysqlBusiness  Mysql `mapstructure:"mysql-business" json:"mysql_business" yaml:"mysql"`
```

#### 增加DB控制器代码

touch initialize/business.go

vim initialize/business.go

```
package initialize

import (
	"github.com/flipped-aurora/gin-vue-admin/server/global"
	"gorm.io/driver/mysql"
	"gorm.io/gorm"
	"gorm.io/gorm/schema"
	"fmt"
)

func  BusinessDb() (gormDb *gorm.DB, err error) {
	m := global.GVA_CONFIG.MysqlBusiness
	//dns := "root" + ":" + "mqzhifu" + "@tcp(" + "8.142.177.235" + ":" + m.Port + ")/" + "test" + "?" + m.Config
	dns := m.Username + ":" + m.Password + "@tcp(" + m.Path + ":" + m.Port + ")/" + m.Dbname + "?" + m.Config
	fmt.Println("mysql link info: " + dns)
	mysqlConfig := mysql.Config{
		DSN:                       dns,   // DSN data source name
		DefaultStringSize:         191,   // string 类型字段的默认长度
		DisableDatetimePrecision:  true,  // 禁用 datetime 精度，MySQL 5.6 之前的数据库不支持
		DontSupportRenameIndex:    true,  // 重命名索引时采用删除并新建的方式，MySQL 5.7 之前的数据库和 MariaDB 不支持重命名索引
		DontSupportRenameColumn:   true,  // 用 `change` 重命名列，MySQL 8 之前的数据库和 MariaDB 不支持重命名列
		SkipInitializeWithVersion: false, // 根据版本自动配置
	}
	diyConfig := &gorm.Config{DisableForeignKeyConstraintWhenMigrating: true, NamingStrategy: schema.NamingStrategy{SingularTable: true}}
	db, err := gorm.Open(mysql.New(mysqlConfig), diyConfig)
	fmt.Println("gorm.Open er  info:", err)
	if err != nil {
		return nil, err
	}
	global.GVA_DBList = make(map[string]*gorm.DB)
	global.GVA_DBList["business"] = db
	return db, nil
}

```

#### main添加启动新DB的入口

vim main.go

```
initialize.BusinessDb()
```

#### 自带的时间格式是基于DB datetime类型，而日常使用的不依赖DB，就是个INT值

touch model/my\_model.go

vim model/my\_model.go

```
package model

import "gorm.io/gorm"

type DIY_MODEL struct {
	ID        uint           `gorm:"primarykey"` // 主键ID
	CreatedAt int            // 创建时间
	UpdatedAt int            // 更新时间
	DeletedAt gorm.DeletedAt `gorm:"index" json:"-"` // 删除时间
}

```

#### 打开跨域

vim initialize/router.go

打开跨域注释：

> Router.Use\(middleware.Cors\(\)\)

## 自动生成页面代码

#### 导入常量值

> 自动生成页面时，有些字段时是枚举，为了减少代码量，从原项目中拿到该值，并直接生成sql query，直接插入到DB中

请求接口拿到sql:

> http://127.0.0.1:1111/tools/const/init/db

导入 gva DB 中

```
delete from sys_dictionaries where id >= 10;
delete from sys_dictionary_details where sys_dictionary_id >= 10 
```

#### 创建一个包：

> 所有的页面，得给一个公共包

1. 系统工具\-\>自动化package\-\>创建一个包:op 运维 运维日常维护使用的一些表格数据工具
2. 系统工具\-\>生成代码器

#### 创建根菜单：

> 目前先把所有的页面统一放在这个新的菜单中，后续再考虑如何放置位置

超级管理员\-\>菜单管理\-\>新建根菜单\-\>运维\-\>op

给新创建的根菜单，添加一个索引页面，用于包含子菜单的代码:

> touch view/op/index.html

新建子菜单：

> view/op/statisticsLog.vue

#### 自动创建代码：

系统工具\-\>代码生成器\-\>数据库名\-\>表名\-\>生成代码

需要创建的页面：

```
statistic_log
project
instance
server
```

调整字段的时候注意下：tinyint 它是给转成了bool 得手动改成int

将刚刚生成的模板文件，转移到指定目录下：

> mkdir web/src/view/op
> 
> 
> mv web/src/view/statisticsLog/\* web/src/view/op

修改model

```
model.DIY_MODEL
```

后端代码修改：

vim server/service/op/statistic\_log.go

replace:

```
global.GVA_DB
global.GetGlobalDBByDBName("business")
```

给角色设置新权限，退出 ，重新登陆即可。

#### 导出DB

/soft/mysql/bin/mysqldump \-uroot \-h8.142.177.235 \-p'\#\#\#\#\#' gva \> /data/www/gva.sql

## 自有安装

> git http://192.168.1.22:40080/frontend/seed\_admin\_vue.git
> 
> 
> git http://192.168.1.22:40080/backend/seed\_admin\_go.git

> 以上是属于空项目安装，下面是：直接下载已经部署好的项目

下载代码：

> chown \-R wangdongyan:\_www /data/www/golang/seed\_admin\_vue
> 
> 
> chown \-R wangdongyan:\_www /data/www/golang/seed\_admin\_go

后端密码：admin 123456

## 安装前端

cd seed\_admin\_vue

> npm install \-\-unsafe\-perm=true \-\-allow\-root

安装不成功的话，换个淘宝的源：

> cnpm install \-\-unsafe\-perm=true \-\-allow\-root

两种方式均可，官方更推荐npm，但速度没有cnpm快

启动:

> npm server

查看下端口号：

lsof \-i:8080

配置文件修改：

```
vim .env.product
>VITE_BASE_API = http://adminapi.seedreality.com
```

vim vite.config.js

> outDir: '../dist'

产出目录,修改编译JS输出目录

## 安装后端

#### 把DB导入到本地：gva.sql

source gva.sql

mod下包，并启动后端:

```
cd seed_admin_go
go mod tidy
go run main
```

修改配置信息：

vim config.yaml

1. DB\-MYSQL配置，一个GVA自带的，另一个连接到业务库
2. 自动生成代码的根目录，root: /data/www/golang

自动生成代码：

1. 先确定放在哪个菜单下，如果是已存在的根目录就忽略，如果是新的根目录，那得新建一个
2. 创建一个子菜单
3. 自动生成的代码，得有个包名\(package\)，这里如果已经存在就忽略，不存在 手动创建一个
4. 系统工具\-》代码生成器\-》数据库名\-》表名\-》下面就根据表的字段生成了table
5. 选择包名
6. 对每个字段进行编辑
    1. 是否添加搜索功能
    2. 它仅支持varchar int ，像：tinyint 他是给转成了bool，这里注意下，改回int
    3. 如有枚举字段，记得要选择具体的常量列表
7. 点击确认后，它会生成前端、后端代码
8. 前端代码在：view/packageName下面，你转移到自己想放的位置，然后修改你上面刚刚创建的子菜单，把这个文件地址复制进去
9. 后端代码略多，不详细写了，具体，你就需要修改：
    1. model/packageName/文件 ，把它的 global.GVA\_MODEL 改成 model.DIY\_MODEL
    2. 如果不是GVA库的表，是业务表， service/packageName/文件 ，把它的 global.GVA\_DB 统一换秘 global.GetGlobalDBByDBName\("business"\)

#### 任务1：

页面修改

#### 删除默认密码/始化按钮

view/login/login.vue

168行：

删除admin 和 123456

#### 换LOGO

#### 删除尾部

#### 个人信息，修改头部 手机号 地址 邮箱 等等

等等吧，把这个开源项目自带的一些东西，都弄掉吧。

任务2：

自己创建一个页面，如：声网回调这张表

两个任务的核心是熟悉这个后台项目的代码
