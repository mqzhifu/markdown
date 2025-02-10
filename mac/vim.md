# vim

## vim

vim的配置，真是可以用\<哲学\>来形容了，因为是最早的文本IDE，牵扯的配件实在是太TMD多了，如下：

1. nerdtree 目录结构树，分栏显示
2. php语法高亮
3. php自动补全
4. bundle 管理插件的工具
5. 函数点击跳转/函数结尾跳转到函数顶部
6. 自动补全小括号、大括号等
7. 文件中的函数列表
8. 代码/函数 折叠
9. 底部\-状态栏
10. ....\(太多了\)

反正吧，挺有意思，比如：高亮是高亮，函数提示是函数提示，全要单独配置。可能是被一些高级IDE惯坏了，或者不是早期程序员，也不写C\+\+，对于纯手工配置真的是挺头疼的。

nerdtree

> wget http://www.vim.org/scripts/download\_script.php?src\_id=17123 \-O nerdtree.zip

```
mkdir nerdtree
mv nerdtree.zip nerdtree
cd nerdtree
unzip nerdtree.zip
mkdir -p ~/.vim/{plugin,doc}
cp plugin/NERD_tree.vim ~/.vim/plugin/
cp doc/NERD_tree.txt ~/.vim/doc/
```

vim中输入:NERDTree

若想打开vim自动进入NERDTree,则在~/.vimrc

touch ~/.vimrc

文件中添加下面内容

```
autocmd VimEnter * NERDTree  
wincmd w
autocmd VimEnter * wincmd w
let NERDTreeWinPos=1
autocmd bufenter * if (winnr("$") == 1 && exists("b:NERDTreeType") &&b:NERDTreeType == "primary") | q | endif

map <F2> :NERDTreeMirror<CR>
map <F2> :NERDTreeToggle<CR>
```

## vim 状态栏

powerline\-status

查看py是否已安装

> python \-\-version

安装一个py的插件pip

> easy\_install pip

安装 powerline

> pip install powerline\-status

查看安装目录

> pip show powerline\-status

vimrc 中添加

```
set rtp+=/Library/Python/2.7/site-packages/powerline/bindings/vim
set laststatus=2
set t_Co=256
```

## PHP高亮插件

下载 php.tar 包，解压

```
syntax/php.vim
mv syntax ~/.vim/
vim ~./.vimrc
syntax  on
```

autocmd FileType php set omnifunc=phpcomplete\#CompletePHP

```
set autoindent  #自动缩进，回车后继承上一行的缩进
set ts=4        #设置tab 的长度为4
set expandtab   #设置tab 为空格

#以下是 自动补齐
inoremap ' ''<ESC>i
inoremap " ""<ESC>i
inoremap ( ()<ESC>i
inoremap [ []<ESC>i
inoremap { {<CR>}<ESC>O

set ignorecase  #搜索时忽略大小写

```

## 用户/组

useradd groupadd 都没有

\#查看所有组列表

> dscl . \-list /Groups

\#查看所有组的ID

> dscl . \-list /Groups PrimaryGroupID

\#创建一个组rsync,但不指定ID\( 参数PrimaryGroupID将查不到 \)

> dscl . \-create /groups/rsync

\#创建一个组rsync，并指定ID为 888

> dscl . \-create /Groups/rsync gid 888

\#删除一个组rsync

> dscl . \-delete /Groups/rsync

将某用户从某组中删除

> dscl . \-delete /Groups/某组 GroupMembership 用户名

\#查看所有用户

> dscl . \-list /Users

\#创建一个用户rsync

> dscl . \-create /Users/rsync

\#将某用户添加到某组中

> dscl . \-append /Groups/rsync GroupMembership rsync
