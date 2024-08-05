# viper

解析/读取 配置文件

#### 主要功能：
- 解析功能，所有的配置信息均可以解析成一个 map 结构
- 支持的文件类型格式较多
- 支持的数据源较多

支持文件格式类型：

1. json
2. yaml
3. toml
4. hcl
5. env
6. cmd
7. Java properties

> json yaml toml java\-properties这个就不我解释了。

#### 数据源读取方式

1. 本地文件
2. 指令行参数（从指令行中读取启动参数，使用 flags 映射）
3. 系统环境变量（从LINUX系统环境变量中取值）
4. 已打开的IO流
5. 远程直接读取数据流（etcd/consul）

#### watching

监听配置文件变化，动态加载。这个功能挺方便

#### 优点

1. 整合了的像：系统变量、指令行数据等松散的数据
2. 帮忙解析特殊格式类型的配置文件体。如：yaml toml json
3. 所有数据读取完成后，在一个统一的类中，统一管理使用。
4. 动态监控文件变化
5. 读取远程配置文件

#### 缺点

好像不支持多配置文件，如：一个目录下有N个配置文件都要加载。不过，可以通过多实例，每一个实例是一个配置文件来解决

# Casbin

用户权限控制

优点：

1. 支持多语言（不依赖某一种语言，更像是一种算法）
2. 有论文（有几个人，专门研究权限控制这一领域）
3. 支持RBAC 、ABAC 多种算法
4. 自定义了一种 PML 语法
5. 自定义了PERM组织关系

缺点：

1. 项目复杂度变高
2. 加大学习、使用、配置等 维护成本

**PML**:一种基于解释器的Web服务访问控制策略语言

**PERM:**

|英文    |中文     |描述                                        |
|--------|---------|--------------------------------------------|
|policy  |策略     |定义policy.cvs文件中的字段顺序              |
|effect  |效果/影响|定义组合了多个 Policy 之后的结果, allow/deny|
|request |请求     |访问请求, 也就是谁想操作什么                |
|matchers|匹配器   |确定policy与request的比对结果               |
|role    |角色     |定义用户角色时使用                          |

> 讽刺的是，它只支持一个p，如果再有个P2不支持...

**大体上的实现流程**

两个配置文件：perm.conf \+ policy.cvs

perm.conf:定义整个权限控制算法规则的配置信息

policy.cvs:是具体某个用户有哪个些权限

PML用来解析两个配置文件

流程图，如下：

1. 管理员 配置 perm.conf \+ policy.cvs
2. 程序员 启动casbin程序，PML加载两个配置文件
3. 用户请求资源，casbin根据两个根本文件进行计算

perm.conf:

```
# Request definition
[request_definition]
r = subject, obj, act

# Policy definition
[policy_definition]
p = subject, obj, act
p2 = subject, act

# Policy effect
[policy_effect]
e = some(where (p.eft == allow))

# Matchers
[matchers]
m = r.subject == p.subject && r.obj == p.obj && r.act == p.act

[role_definition]
g = _, _

```

policy.cvs

```
p, alice, data1, read
p, bob, data2, write
```

perm.conf是基础语法定义，基本上一次定义基本不变，而policy.cvs是对perm.conf的数据补充，而policy.cvs是经常变的，而且要存DB的。

**分析PERM**

request:感觉是对资源的层级定义，可以定义N级，如：

```
order-coupon-add
public-user-login
private-logn-add
```

也可以映射到的URL的请求地址中

```
/order/coupon/add
/public/user/login
/private/logn/add
```

作用：定义资源，并做成层级结构，方便拆分与管理，具体如何定义看使用方，如：

1. 模块/模型/动作
2. 用户/模型/动作

> 基本上3层结构能满足大部分的需求了

**具体流程**

1. 管理员 先定义 资源，即：资源是几层，给每层加上tag\(subject obj act\)

> request= subject obj act

这就是定义了一个三层的资源

> 定义好的层数其实也是请求参数，即：当过来一个请求，你调用casbin方法验证，验证的这个方法的参数个数

1. 然后是，赋值之前的前置工作，得给policy.csv 做下字段说明,好让PML能解析成功

> p = act,obj,subject

这就是说在读取policy.cvs 文件的时候，里面的字段是按照这种顺序存储的

1. 然后，是给这个资源赋值，即：\<谁有这样的权限\>，\(这个保存在policy.csv中\)
    1. p , edit,user,zhangsan
    2. p , del ,user,zhangsan

> 以上：张三 访问user资源，有编辑和删除的功能

> 以上就定义好了：\<资源\>和\<谁有这样的权限\>

1. 定义两者的匹配公式

> \(这里直接用等号判断不就结了，还搞个匹配公式，真是麻烦\)

例子：

> m = r.subject == p.subject && r.obj == p.obj && r.act == p.act

这是最简单的，就是等号判断，既然定义了三层资源结构，那就直接与配置好的\<谁有这样的权限\>一一做判断.

> matching公式最后匹配的结果保存在：p.eft中

至此，最简单的ACL就可以了，下面是复杂的设计

1. 定义多条相同的rule，policy.csv添加如下：
    p , edit,user,zhangsan,allow
    p , edit,user,zhangsan,deny
    定义了2条相同的rule，但是返回是不一样的
    修改下perm.conf
    p = subject, obj, act ,eft
    这样如果r给出：edit,user,zhangsan ,matching将会得出2个结果，allow 和deny
    然后policy\_effect就登场了
2. policy\_effect设置成，当matching匹配出多个结果的时候，再进行一次匹配，得出最终结果
    它其实规定死了，就4个表达式：
    some\(where \(p.eft == allow\)\)
    \!some\(where \(p.eft == deny\)\)
    some\(where \(p.eft == allow\)\) && \!some\(where \(p.eft == deny\)\)
    priority\(p.eft\) || deny
    subjectPriority\(p.eft\)
3. 角色，上面都是基于传统ACL，得加入角色转换成RBAC,修改：perm.conf

```
[role_definition]
g = _, _
[matchers]
m = g(r.sub, p.sub) && r.obj == p.obj && r.act == p.act
```

> g = \_, \_ = 用户,组名

修改policy.cvs:

zhangsan,admin

g\(r.sub, p.sub\): 将zhangsan转换成admin

> 这里有个有趣的：访问的时候即可以使用组名也可以使用用户名

g = \_, \_, \_

> 这个也挺有意思:用户/组/域，也就是它的角色支持3级，如：用户/组/商户

最终的流程是

1. 第一次，需要管理员 注册request资源，配置好policy字段顺序，然后写好match公式，
2. 基本上就不用动了，其它的都是动态修改policy.cvs

**总结**

它实现了：将各个模块完全剥离出来，各自玩各自的，通过配置文件再组合起来。减少耦合，增加扩展性，尤其像：支持用户ID和组ID同时匹配、角色支持3级。

> 感觉，基本上这东西跟apahc nginx 的rewrite 差不多。

结果：

1. 忽略掉学习成本，也就是已经熟悉了该模式，并能给别人讲明白原理
2. 自己能熟悉使用该类库
3. 可以使用，挺方便，主要不用写代码了...扩展性也好

