# 新mac安装

## U盘安装

1. 插上U盘
2. 打开mac自带disk工具
3. U盘格式化成：Mac os 扩展\(日志式\)
4. 打开官网地址

> https://support.apple.com/zh\-cn/HT201372

1. 下载os镜像
2. 制作U盘引导\+OS镜像

> sudo /Applications/Install\\ macOS\\ Catalina.app/Contents/Resources/createinstallmedia \-\-volume /Volumes/MyVolume

> sudo /Applications/Install\\ macOS\\ Ventura.app/Contents/Resources/createinstallmedia \-\-volume /Volumes/MyVolume

1. command \+ R ,进入恢复模式，上面菜单栏，打开：允许外部安装系统
2. options \+ R ,选择那个U盘
3. 格式化掉MAC 硬盘的所有内容，注：给硬盘起个新名字，不然他就自动带个：data

## OS初始硬盘容量

目前硬盘已使用容量：11.05 \+ 5.5

## OS基础设置

1. 系统设置 \-\> 显示器 排列 \-\> 打开显示器镜像
2. 系统设置\-\>时间与日期 \-\> 时间改成24小时，并显示日期
3. 设置 \-\> 键盘 \-\> 快捷键：\(修改快捷键\)
    1. caps键切换输入法
    2. 聚集 \-\> OP \+ SPACE （切换聚集）
    3. 输入法 \-\> cmd \+ space\(切换输入法\)
    4. 调度中心 \-\> ctrl \+ d（显示桌面）
    5. 服务 \-\> cmd \+ shift \+ f （快速打开finder）
4. 系统设置 \-\> 键盘 \-\> 输出法 \-\> 删除掉拼音
5. 系统设置 \-\> 软件更新 \-\> 关闭mac 自动更新
6. 系统设置 \-\> 安装性与隐私 \-\> 防火墙 \-\> 关闭
7. 系统设置 \-\> 节能 设置锁屏幕 时间
8. 点击状态 栏 上的电池LOGO 显示电池 百分比
9. 系统设置 \-\> 安全性与隐私 \-\> 通用 \-\> 选择“任何来源”

> 如果没有'任何来源选项'，执行：sudo spctl \-\-master\-disable

## 基础软件安装

1. app store
    1. thor
    2. the unarchier
    3. wechat
    4. lemon
    5. qq
    6. lark
    7. 万年厍
    8. foxit
    9. 参考另外一个[文�]()�
2. 本地文件安装
    1.搜狗五笔输入法
    > OS设置:键盘\-\>删了拼音输入法
    1. hyper switch
    2. 阿里云盘
    3. cDock
    4. foobar
    5. HandShaker
    6. IINA
    7. iTerm
    8. openmtp
    9. Paste
    10. SecureCRT
    11. google chrome
    12. element\-key
    13. 迅雷

## karainer\-element

更换的健位：

|原健|更换|
|----|----|
|fn  |cmd |
|cmd |fn  |

选择complex modifications

新建一个 add rule

选择 import

|原健  |更换         |
|------|-------------|
|return|open         |
|use f2|rename       |
|del   |move to trash|

## 设置finder

1. 显示 \-\>路径\(目录\)栏 状态栏
2. 自定义工具栏 加上：前进后退 路径显示 新建文件 删除
3. 偏好设置：ctrl \+ ,
    1. 配置一下左侧栏目显示
    2. 显示所有文件扩展名
    3. 搜索改成：只搜索当前文件夹子
    4. 在桌面上 显示硬盘
4. 将data film 目录拖到边栏中去

## 开启ROOT

1. 设置 用户 登陆选项\(左下角\) 网络账号服务器 加入 打开目录实用工具 上方菜单 编辑 启用root 设置密码
2. spotlight 直接输入：目录实用工具 更快

## 关闭csrutil（SIP）

1. 重启 ， command \+ r ,进入恢复模式
2. 上方菜单中 开启一个shell

> csrutil disable

1. 重启

## 创建能用文件目录

mount \-uw /

mkdir /data

## xcode & command tools

下载地址：

> https://developer.apple.com/download/all/?q=xcode

OS版本与xcode对应版本

> https://xcodereleases.com/

慢长的下载吧。。。。10G\+400M\+

解压xcode.xip ,插入到 application 中，双击，它还得安装一会

安装command tools

## brew

> /bin/zsh \-c "$\(curl \-fsSL https://gitee.com/cunkai/HomebrewCN/raw/master/Homebrew.sh\)"

这个可以，但依赖GIT，建议先安装好xcode 和command tools

安装过程中，风扇好响 。。。且又是一排 红字，大概意思是：以ROOT运行brew非常危险（去你妹的，老子的电脑，我想怎么用ROOT就怎么用！）

```
brew update
brew install wget
brew install tree
brew install ffmpeg
```

## 设置cDock

安装 cDock

系统设置 \-\> 程序坞 \-》 关闭 显示最近打开的APP

## 翻墙软件

https://hkscloud.men/\#/login

安装 clashX

## alfred

[请点击]()

## github\-ssh

ssh\-keygen \-t rsa \-C "mqzhifu@sina.com"

却是github setting ssh 中添加一个公钥

## item2

具体就不写了，参照另外一个[文档]()

记得配置login.sh os\_startup.sh

## vim

[请点击]()

## 设置开机自启

thor

magent

Hyper switch

## 关闭spotlight索引

sudo launchctl unload \-w /System/Library/LaunchAgents/com.apple.Spotlight.plist

sudo launchctl load \-w /System/Library/LaunchAgents/com.apple.Spotlight.plist

sudo launchctl unload \-w /System/Library/LaunchDaemons/com.apple.metadata.mds.plist

sudo launchctl load \-w /System/Library/LaunchDaemons/com.apple.metadata.mds.plist

关闭自动索引

> sudo mdutil \-a \-i off

删除索引

> sudo mdutil \-a \-E

重新打开

> sudo mdutil \-a \-i on

重建索引

> sudo mdutil \-E

## 最后硬盘空间

折腾完剩下：422个G

chown \-R $\(whoami\) $\(brew \-\-prefix\)/\*

## 移动硬盘被坑

移动硬盘格式不同，WIN下是NTFS，MAC下是HFS\+EXFAT，可以读，但不能写，如果想要写ntfs需要额外最少花39员买个软件，且39元只能支持一台设备、读写速度慢、无法保证MAC更新版本继续使用。我TMD也是贱，非要自己尝试破解，结果1.8T的数据全丢，不少都是N年积累下来的禁播电影，还有海伟的高清美剧，操了！

先不考试丢的数据了，查查解决办法吧

具体方法：

1、MAC下磁盘工具\-\>抹除\-\>再分区成exFat

2、WIN下执行

> chkdsk F: /F

两端都试了下没问题，多了3个文件夹子

> .fseventsd
> 
> 
> .Spotlight\-V100
> 
> 
> FOUND.000

总结

之前是MAC下兼容WIN

现在改成WIN去兼容MAC

WIN下超简单，一条指令，MAC上就得花钱，而且exFat好像还是microsoft 发明的，另外，就算用付费软件，且最高VIP付费，MAC写NTFS的速度依然慢，且有丢数据风险 ，NND，SB的苹果。

虽然数据丢了，但丢就丢了吧，有些不怎么看，还能下的，再重新找找资源，下不了的就算了。当年手杀熊猫算是熟悉了WIN的基本结构，不停的重做系统搞坏硬盘，算是熟悉了WIN的硬盘操作，MAC下也得损失点，不然哪里来的熟悉呢~当是学费吧。

另外，WIN下麻烦点，每次都得执行一下指令才能使用，但是总比花钱要强，而且，现在大部分时间使用MAC办公，经常传东西也是MAC ，效果更好些。
