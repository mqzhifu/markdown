# 新mac 开发环境

## 配置自带SHELL

\(如果直接用on my zsh ,以下：不用配置了\)

指令行提示符：

> export PS1="\[\\e\[32m\]\[\\u@\\h \\W\]$\[\\e\[m\] "

添加：ll指令\+字符集

> touch ~/.bash\_profile
> 
> 
> vim ~/.bash\_profile

```
export LANG="zh_CN.UTF-8"
export LC_ALL="zh_CN.UTF-8"

alias ll='ls -alF'
alias la='ls -A'
alias l='ls -CF'

export PATH=$PATH:/var/root/go/bin
```

> source ~/.bash\_profile

## 添加：创建文件的小脚本工具

touch /usr/local/bin/create\_file.sh

chmod 744 /usr/local/bin/create\_file.sh

vim /usr/local/bin/create\_file.sh

```
#/bin/bash
#出错即立刻停止，保证调用的高级语言能快速捕捉到
set -e

NEW_FILE=$1
FILE_CONTENT=$2
if [ ! -n "$1" ] ;then
        echo "file name is empty~";
        exit
fi

if [ -f "$NEW_FILE" ];then
        echo "错误：文件已存在，请不要重复创建"
        exit
fi

echo "new file: "$NEW_FILE" , ok. " ;
`touch $NEW_FILE`
#`chown root:wheel $NEW_FILE`
`chmod 744 $NEW_FILE `

if [ -n "$2" ] ;then
        echo "file content: "$FILE_CONTENT
        echo $FILE_CONTENT > $NEW_FILE
fi

```

## 创建目录

mkdir /data/golang

mkdir /data/php

mkdir /data/python

mkdir /data/cplus

mkdir /data/mysql

mkdir \-p /data/logs/golang

mkdir /data/bak

mkdir /data/cicd

mkdir \-p /data/install/zip

mkdir \-p /data/install/src

## sucureCRT

得先破解一下，它的安装包里有说明

然后，创建3个连接

192.168.1.21 （22 seed seed1234）

8.142.161.156\(6655 seedar ~\!@\#$QAZwsxedc \)

8.142.177.235\(\)

## golang

可以用brew install golang ，但是它这个版本略低，还是去官网下载吧

> https://go.dev/dl/go1.18.4.darwin\-amd64.pkg
> 
> 
> https://go.dev/dl/go1.18.4.darwin\-amd64.tar.gz

一个是自动安装包，一个是需要手动安装的包

我用的是安装包，完成后，会自动添加到env\_path 中

如果是tar包，也可以自己添加，vim /etc/profile

> export PATH=$PATH:/var/root/go/bin

自动安装包的目录位置：

> /usr/local/go

安装包的环境变量位置:

> /etc/paths.d

配置下go环境变量：

```
go env -w GO111MODULE=on
go env -w GOPROXY=https://goproxy.cn,direct
```

go env 查看下

测试下：

```
cd /data/www/golang/zgoframe
git clonet git@github.com:mqzhifu/zgoframe.git
go mod tidy
go get -u github.com/swaggo/swag/cmd/swag@v1.7.9
go install github.com/swaggo/swag/cmd/swag@v1.7.9
go run main.go
```

## goland

激活：

```
0.0.0.0 account.jetbrains.com
idea.lanyus.com
```

改下目录权限

> chown \-R wangdongyan:\_www /data/www/golang

需要配置下：GO的路径、编译执行参数等

```
偏好设置->Go->Go moudles->environment中添加：
GOPROXY=https://goproxy.cn;GO111MODULE=on

偏好设置->Go->Go Root ，/usr/local/go

```

设置好后，到项目中的.mod 文件中，下载试试

引入配置文件

添加插件：

toml

vue

## redis

创建目录

```
mkdir /soft/redis
mkdir /soft/redis/run
mkdir /soft/redis/data
```

> brew install pkg\-config

brew与item好像共用zsh，brew因为是非root使用，而item2我每次是root进去，这两用户的权限冲突

> sudo chown \-R $\(whoami\) /usr/local/share/doc /usr/local/share/man /usr/local/share/man/man1 /usr/local/share/zsh /usr/local/share/zsh/site\-functions

之后root现变一下权限就行了

> chown \-R $\(whoami\) /usr/local/share/zsh

```
wget https://download.redis.io/redis-stable.tar.gz
tar -zxvf redis-stable.tar.gz
cd redis-stable
vim README.MD
make 
make install    #默认是安装在:/usr/local/bin下
make install PREFIX=/soft/redis #这里也可以加上前缀，所有的redis指令就会被编译到指定目录
```

启动，测试一下

> /soft/redis/bin/redis\-server

```
cp /data/install/src/redis-stable/redis.conf /soft/redis
vim /soft/redis/redis.conf
```

配置主要修改：

```
port 6370
daemonize yes
requirepass 1234567890
pidfile /soft/redis/run/redis_6379.pid  
dir /soft/redis/data  
logfile "/soft/redis/log/redis.log"  
```

启动，测试一下

> /soft/redis/bin/redis\-server /soft/redis/redis.conf

用cli 测试一下

> /soft/redis/bin/redis\-cli \-p 6370

## NGINX

发现还有个鬼东西

xcode\-select \-\-install

好像安了这个鬼东西后，能用GCC

然而 ，失败，上APP STORE 发现 X\-CODE可以随便安装，管他呢，先安一个，然而 11G ，真可怕 ！先下着吧，处理其它 的吧

## apache

> 下载官方包再编译安装失败，用mac系统里自带的吧

vim /etc/apache2/httpd.conf

添加如下：

```
DocumentRoot "/data/www"
<Directory "/data/www">
```

apache \-k restart

127.0.0.1

试了下OK

apache整合PHP

找如下代码，打开注释，加载PHP模块

> LoadModule php7\_module libexec/apache2/libphp7.so

测试一下是否成功

cd /data/www

touch a.php

chmod 777 a.php

vim a.php

```
<?php
phpinfo();
```

> 127.0.0.1/info.php

虚拟目录

```
<VirtualHost *:80>
ServerName local.test.com
DocumentRoot /data/www
</VirtualHost>

```

增加本地HOST映射

vim /etc/hosts

> 127.0.0.1 local.static.com local.api.com local.test.com local.admin.com local.agent.com local.mqadmin.com local.house.com

配置URL重定向

```
vim /etc/apache2/httpd.conf
LoadModule rewrite_module libexec/apache2/mod_rewrite.so
apachectl -k restart

AllowOverride None
修改
AllowOverride All
```

## mysql

UI安装法

> https://dev.mysql.com/doc/mysql\-osx\-excerpt/8.0/en/osx\-installation\-pkg.html

下载包

> https://dev.mysql.com/downloads/mysql/

```
tar -zxvf mysql
mv mysql /soft/mysql

mkdir /data/mysql
mkdir /data/mysql/master_3306
mkdir /data/mysql/init_db
chown -R _mysql:_mysql /data/mysql
chown -R _mysql:_mysql /soft/mysql

cd /soft/mysql/bin
./mysqld --initialize --user=_mysql --basedir=/soft/mysql --datadir=/data/mysql/init_db

cp /data/mysql/init_db/* /data/mysql/master_3306
chown -R _mysql:_mysql /data/mysql/master_3306

```

配置文件

> touch /etc/my.cnf
> 
> 
> vim /etc/my.cnf

启动:

> /soft/mysql/bin/mysqld\_safe &

登陆：

> mysql \-uroot \-p 临时密码

新版本的密码加密格式变了，会导致MYSQL客户端连不上，改回到旧版的

> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql\_native\_password;

修改下新的密码

> alter user 'root'@'localhost' identified by 'mqzhifu';

mysql快速关闭的工具

```
create_file.sh /usr/local/bin/stop_mysql.sh '/soft/mysql/bin/mysqladmin -uroot -pmqzhifu shutdown'
```

mysql快速启动的工具

```
create_file.sh /usr/local/bin/start_mysql.sh '/soft/mysql/bin/mysqld_safe &'
```

mysql快速登陆的工具

```
create_file.sh /usr/local/bin/login_mysql.sh '/soft/mysql/bin/mysql -uroot -pmqzhifu seed_test'
```

## php\-redis扩展

```
wget http://pecl.php.net/get/redis-5.3.2.tgz  
tar redis-5.3.2.tgz  
cd ../src_bag/redis-5.3.2
```

phpize

发现少autoconf

brew autoconf install

重新phpize

报错：/usr/include/php/main/php.h

一直报这个错误，实在没办法了，试下安装xcode指令行工具

xcode\-select \-\-install

不能安装该软件，因为当前无法从软件更新服务器获得

这个狗东西，安装不了，只能去网站下载了

> https://developer.apple.com/download/more/

改一下系统头库

```
cd /usr
sudo ln -s /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.15.sdk/usr/include /usr/include
```

失败,换个方法尝试

./confiere ，不加 php\-config

OK了

make & make install

php扩展默认生成目录

> /usr/lib/php/extensions/no\-debug\-non\-zts\-20180731/

添加php redis扩展

```
chmod 744 /etc/php.ini
vim /etc/php.ini
extension=redis.so
apachectl -k restart
```

试一下，是否成功

> local.test.com/info.php

OK,算是MAC下,第一次安装php扩展，好麻烦

## zip扩展

cd /install/tar\_bag

wget http://pecl.php.net/get/zip\-1.19.1.tgz

tar \-zxvf zip\-1.19.1.tgz

mv zip\-1.19.1 ../src\_bag/

cd ../src\_bag/zip\-1.19.1/

phpize

Please reinstall the libzip distribution

wget https://libzip.org/download/libzip\-1.5.2.tar.gz

没速度 ，NND，先不浪费 时间 了，修改PHP，跳过这个扩展

## JDK

先到官网下一个mac 版的JDK 包 dmg

一路下一步安装完成

```
cd ~
touch .bash_profile
vim
JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk-15.0.1.jdk/Contents/Home
PATH=$JAVA_HOME/bin:$PATH:.
CLASSPATH=$JAVA_HOME/lib/tools.jar:$JAVA_HOME/lib/dt.jar:.
export JAVA_HOME
export PATH
export CLASSPAT

source ~/.bash_profile
```

## jmeter

http://jmeter.apache.org/download\_jmeter.cgi

```
vim ~/.bash_profile
export JMETER_HOME=/Users/wangdongyan/Downloads/apache-jmeter-5.3
export PATH=$PATH:$JMETER_HOME/bin
```

source ~/.bash\_profile
