# mac dir 说明

/Application

/Library/Application

/System/Applications

/System/Volumes/Data/Applications

/Users/wangdongyan/Applications

/Users/wangdongyan/Library/Application Support

/Volumes

## unix/linux 传统目录

/bin :日常操作系统的命令 ls cd rm

/sbin:日常管理系统的命令 fdisk ifconfig ping

/usr:3方类库

bin:后期安装的一些可执行脚本命令

sbin:后期安装的一些可执行脚本命令

local/bin:用户自己编译的软件

/etc:所有配置文件

/dev:硬件设备，如：硬盘驱动

/tmp:临时文件

/var:可能产生变化的东西，如：日志文件

## mac os

```
Applications 应用程序目录，默认所有的GUI应用程序都安装在这里；
Library 系统的数据文件、帮助文件、文档等等；
    Application
    Application Support:三方应用插件
    Caches
    Documentation
    Fonts
    
Network 网络节点存放目录，不过我的电脑里没这个目录呢
System 他只包含一个名为Library的目录，这个子目录中存放了系统的绝大部分组件，如各种framework，以及内核模块，字体文件等等。
    Library
        cache:好大....
        fonts:
        Sounds:
        desktop pic:系统自带的桌面图片
        openssl
    Applications:系统自带的软件，如：app store、books、Calculator、music、photo 等
    
Users 存放用户的个人资料和配置。每个用户有自己的单独目录。等同于linux下/home
    wangdongyan
        Applications
        Desktop
        Documents
        Downloads
        Library
            Application Support:三方应用插件
        Movies
        Music
        Pictures
        Public 你可以把需要与其它用户共享的文件放在这个目录中，默认状态下，这个目录可以被其它所有用户访问
        各种软件的配置文件、缓存文件
        
Volumes 文件系统挂载点存放目录。有时候安装一些非APP Store下载的包，首次会挂载到这里
cores 内核转储文件存放目录。当一个进程崩溃时，如果系统允许则会产生转储文件。
private 里面的子目录存放了/tmp, /var, /etc等链接目录的目标目录。

```
