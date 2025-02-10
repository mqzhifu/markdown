# item2

比mac下自带的terminal更人性化些，好用！

## 改变home end 行首行尾了

打开 key

add hex key

0x01

0x05

可恨的ctrl \+ c 中断进程，也不能用了，想办法解决

stty \-a ，查看当前iterm2工具 键盘组合键 转换成的 \<发送值\>

失败......

ps:正常不安装，my\-zsh，上面需要改，这了后，发现有冲突！！！

打开全局快捷键

> sys setup \-\> keys \-\> hotKey \-\> ctrl\+空格

account \-\>添加自启

也可以设置个背影图

## 可选\-关闭鼠标选中复制

> profiles\-\>default\-\>terminal\-\> enable mouse reporting

关闭后，改用ctrl\+c复制，另外，鼠标滚动也失效，得重新配置下

## 高级配置

echo $0

正常返回是：\-zsh，如果不是，执行：chsh \-s /bin/zsh

默认会在:

/bin/zsh

/etc/shells 文件中，多一项：/bin/zsh

## 每开启一个shell，即执行下面指令

touch /Users/wangdongyan/Documents/login.sh

chmod 777 /Users/wangdongyan/Documents/login.sh

vim /Users/wangdongyan/Documents/login.sh

```
expect -c '
spawn sudo su - root
expect ":"
send "mqzhifu\r"
interact                                                    
'
```

改下配置

> profiles \> general \-\> custom shell \-\> /Users/wangdongyan/Documents/login.sh

## 开机自动执行脚本

touch /Users/wangdongyan/Documents/mount.sh

chmod 777 /Users/wangdongyan/Documents/mount.sh

vim /Users/wangdongyan/Documents/mount.sh

开机自启执行脚本:

```
expect -c '
spawn sudo su - root
expect ":"
send "mqzhifu\r"

expect "%"
send "mount -uw /\r"
interact
'
```

system \-\> 用户 \-\> 登陆 \-\> /Users/wangdongyan/Documents/mount.sh

## 开始配置主题

1. 这里有个小插曲：修改 mac 网络 DNS为：8.8.8.8 114.114.114.114
2. vim /etc/resolv.conf ,DNS为：8.8.8.8 114.114.114.114
    两种方式均可

先安装个插件\(翻墙\)

> sh \-c "$\(curl \-fsSL https://raw.githubusercontent.com/robbyrussell/oh\-my\-zsh/master/tools/install.sh\)"

会提示你，是否将此主题设置为默认模式，选择 yes

它会覆盖 ~/.zshrc

ll 一下，配色真的很爽

这个主题还有插件功能，设置开启一下：

vim ~/.zshrc

```
#这行注释打开，不然 local/bin 下的文件找不到
export PATH=$HOME/bin:/usr/local/bin:$PATH
#改一下主题配置
ZSH_THEME="agnoster"

plugins=(
  git
  bundler
  dotenv
  rake
  rbenv
  ruby
)

```

ll一下，发现：指令行的样式~乱码，开始下载字体

> https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fpowerline%2Ffonts%2Fblob%2Fmaster%2FMeslo%2520Slashed%2FMeslo%2520LG%2520M%2520Regular%2520for%2520Powerline.ttf

双击安装

更新设置

> profiles \> default \-\> text \-\> meslo LG M for powerline

优点

1. 支持快捷直接切换
2. 文件夹直接给上色了
3. 提示行里的目录也有配色
4. 提示行里，支持git
5. 命令\+tab ，列出所有文件的时候，居然还支持选择\-\>选中\-\>补全

发现汉字乱码

```
vim ~/.zshrc
export LC_ALL=en_US.UTF-8
export LANG=en_US.UTF-8
```

查看提示符

```
echo $PS1 
echo $PROMPT
```

提示符中的用户名/host/目录，太长，修复下

```
vim ~/.zshrc
DEFAULT_USER="root"
```

这里因为我每个shell窗口开启后，会自动切换到root

也可以换种方式修改

查看当前主题名

echo $ZSH\_THEME

vim /var/root/.oh\-my\-zsh/themes/agnoster.zsh\-theme

搜索 DEFAULT\_USER ,注释掉，prompt\_segment 这一行

整行注释，提示符就为 空了

以上的修改，都比较极端且无脑，下面，试下详细的个性

vim /var/root/.oh\-my\-zsh/themes/agnoster.zsh\-theme

到文件最后，会有如下结构体：

```
build_prompt() {
  RETVAL=$?
  prompt_status
  prompt_virtualenv
  prompt_aws
  prompt_context
  prompt_dir
  prompt_git
  prompt_bzr
  prompt_hg
  prompt_end
}
```

prompt\_status:这一行，就是提示符最前面的icon符号

prompt\_context:这一行，就会把整个username@host 删除掉,也就是第2种方法要注释掉的那一行

只留一个 %n :就是用户名

ompt\_segment black default "%n"

指令行最前面的小icon 闪电符号，查了它的文档，目前知道的有3种符号

> 1、闪电，每条指令前都有，不知道干啥的
> 
> 
> 2、删除，这个是ctrl\+c 会显示，执行错误指令也会显示
> 
> 
> 3、小圆圈，代理后台有启动进程，比如：mysql，但是切换个窗口后，这个小圈就没有了

所以，综合上述所看，这个icon 的功能没啥用，注释了

## 小麻烦

都弄完，重新登陆，开一个新shell，发现报错，遇到点小麻烦，执行如下：

> chmod 755 /usr/local/share/zsh
> 
> 
> chmod 755 /usr/local/share/zsh/site\-functions

或

```
chown -R $(whoami) /usr/local/share/zsh
```

或添加 vim ~/.zshrc

> ZSH\_DISABLE\_COMPFIX="true"

## shell指令行~高亮

输入指令的时候，有时候会出现错的情况

```
cd ~/.oh-my-zsh/custom/plugins  
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git

vim ~/.zshrc 

plugins=(git zsh-syntax-highlighting)
```

## git\-快捷操作

|Alias|Command           |
|-----|------------------|
|gaa  |git add \-\-all   |
|gba  |git branch \-a    |
|gcam |git commit \-a \-m|
|gd   |git diff          |
|gl   |git pull          |
|gp   |git push          |
|gr   |git remote        |
|gt   |git status        |
|gb   |git branch        |
|Alias|Command           |

zsh\-autosuggestions

autojump
