# hexo-博客

## 安装与体验

前置条件，需要已安装过：git 、node 、npm

先安装它的工具\-指令行

```
npm install -g hexo-cli
```

创建一个文件夹并进入：

```
>mkdir /data/www/golang/myblog
>cd /data/www/golang/myblog
```

安装：

```
>hexo init   #初始化，感觉就是下代码(必须得是一个空文件夹)
>npm install #安装，应该安装具体的库 改改配置文件之类的，不确定这一步是否可省略
```

生成静态文件：

> hexo g

开启守护进程:\(根目录下的 public 目录\)

> hexo s

打开网页，看看效果:

> http://localhost:4000/

创建一个篇文章（在 source/\_posts 目录下创建了一个文件）：

> hexo n 'my\-first\-blog'

查看刚刚创建的文章详细内容：

> vim /data/www/golang/myblog/source/\_posts/my\-first\-blog.md

清除 已生成的静态文件\(清空public目录\)：

> hexo clean

重启：\(3个指令的合集：先清空，再重新生成静态文件，最后启动服务\)

> hexo clean && hexo g && hexo s

## 创建分类与tag

创建一个分类:

> hexo new page categories

根据提示符打开文件：

> vim /data/www/golang/myblog/source/categories/index.md

```
type : categories
```

创建一个tags:

> hexo new page tags

根据提示符打开文件：

> vim /data/www/golang/myblog/source/tags/index.md

```
type: "tags"
```

修改配置文件：

> vim \_config.yml

```
category_dir: /categories/
tag_dir: /tags/
```

找一个文章修改，试试效果

> vim /data/www/golang/myblog/source/\_posts/my\-first\-blog.md

```
categories : web前端
tags : vue
```

## 创建\<关于\>页面

hexo new page "about"

vim config

```
menu:
  about: /about/ || fa fa-user
```

## hexo配置文件

vim \_config.yml

```
# Site
title: xiaoz - blog
subtitle: 'life detail'
description: 'Attentively records the good life'
keywords: "technology tech jobs job life worker internet golang php vue js"
author: xiaoz
#如果使用next 这个是旧版
language: zh-Hans 
#如果使用next 这个是新版
language: zh-CN 
timezone: 'Asia/Shanghai'
```

## 注意next版本

5.X版本

> https://github.com/iissnan/hexo\-theme\-next
> 
> 
> start=15.8

7.X版本

> https://github.com/theme\-next/hexo\-theme\-next
> 
> 
> start=7.7K

> 感觉变化挺大的，它直接把URL地址/版本库都给变了，不过新版的github星星略少

为什么我本次安装，会扯出来旧版？

从星星上看，旧版肯定用的人多。URL都换了，有点硬分叉的意思，那么可能就形成：各自版本都有使用的群体。而且旧版人多，多年积累下的文档肯定更多，所以我被节奏，全是旧版的安装方式了...

被坑

1. 文档，两个版本除了配置不一样，查文档时也容易混淆，造成配置不生效的情况
2. 因为两版差异大，第一次配置好的东西基本没用，还得再来一次

## next主题

hexo支持主题更换，官方主题地址：

> https://hexo.io/themes/

感觉这个官方主题地址有点形同虚设，因为：好的主题就那么2~3个，其中最火的就是：next......

这里我也选择:next主题，随大流不一定对，但肯定不会错，先下载主题，这里有两种方法：

方法1：

```
cd /data/www/golang/myblog
git clone https://github.com/theme-next/hexo-theme-next themes/next
```

方法2：

```
cd /data/www/golang/myblog/themes/next
wget https://github.com/theme-next/hexo-theme-next/archive/refs/tags/v7.8.0.tar.gz
tar -zxvf v7.8.0.tar.gz
```

按说方法2更正规，但我看打包的时间 ：2020\-4月，而master的最后提交是：2021\-7月，算了正规的方法放弃了，用git clone吧，麻烦的就是最后统一提交GIT时：得把next的git删除了

修改主站配置:

vim \_config.yml

```
theme: next
```

修改next主题配置：

> vim /data/www/golang/myblog/themes/next/\_config.yml

貌似报错，得安装个：

npm i hexo\-renderer\-swig

hexo s \-\-debug

## next代码块配置

```
codeblock:
  highlight_theme: night bright #高亮 + 黑色背景
  copy_button:
    enable: true #复制按钮
```

## next添加头像

```
avatar:
  url: /images/avatar_fly2.jpeg
  rounded: true #圆形
```

## next社交链接\(侧边栏\)

```
social:
  GitHub: https://github.com/mqzhifu || fab fa-github
  E-Mail: mailto:mqzhifu@sina.com || fa fa-envelope
  Weibo: https://weibo.com/u/1806907257 || fab fa-weibo
```

## 给文章添加结尾话术

我看了看它的脚本：

> vim post.swig

貌似可配置的东西还挺多的，再回到主题配置文件，打开配置项

```
custom_file_path:
  postBodyEnd: source/_data/post-body-end.swig
```

mkdir /data/www/golang/myblog/source/\_data

touch /data/www/golang/myblog/source/\_data/post\-body\-end.swig

vim /data/www/golang/myblog/source/\_data/post\-body\-end.swig

```
<div>
    {% if not is_index %}
        <div style="text-align:center;color: #ccc;font-size:14px;">-------------本文结束<i ></i>感谢您的阅读-------------</div>
    {% endif %}
</div>
```

## 支持流程/甘特图

npm install hexo\-filter\-mermaid\-diagrams \-\-save

修改next主题配置

enable : true

## 添加统计

> vim /data/www/golang/myblog/themes/next/\_config.yml

```
busuanzi_count:
  enable: true
```

旧版的还需要改，新版的就改个配置参数就行了

vim /data/www/golang/myblog/themes/next/layout/\_third\-party/analytics/busuanzi\-counter.swig

```
<script async src="https://busuanzi.ibruce.info/busuanzi/2.3/busuanzi.pure.mini.js"></script>
```

## next添加搜索

```
cd /data/www/golang/myblog/
npm install hexo-generator-searchdb --save
```

vim themes/next/\_config.yml

```
local_search:
    enable: true
```

## 错误

旧版才有这个问题

转到next主题后，发现左侧菜单所有的链接多了一个：%20

vim themes/next/\_config.yml

搜索menu

把每个连接的空格删除了

## 隐藏网页底部无用信息

vim themes/next/layout/\_partials/footer.swig

```
footer:
    powered: false
  theme:
    enable: false
    version: false
```

## 右上角打开GITHUB连接

github\_banner:

enable: true

## next\-设置浏览器 favicon

favicon:

\#small: /images/favicon\-16x16\-next.png

\#medium: /images/favicon\-32x32\-next.png

## 动态背景

cd /Users/mayanyan/data/www/golang/zblog/themes/next

git clone https://github.com/theme\-next/theme\-next\-three source/lib/three

rm \-rf .git

```
three:
  enable: true
  three_waves: true
  canvas_lines: false
  canvas_sphere: false
```

坑B的，3个效果都死难看

cd /data/www/golang/zblog/themes/next

git clone https://github.com/theme\-next/theme\-next\-canvas\-nest source/lib/canvas\-nest

```
canvas_nest:
  enable: true
```

## 分享功能

npm install theme\-next/hexo\-next\-share

npm install \-g grunt \#这个我不确定是不是一定得安装

旧版：安装完成后，配置文件会多出一些配置，改成true 即可

新版：得手动添加一下

```
needmoreshare:
  enable: true
  cdn:
    js: //cdn.jsdelivr.net/gh/theme-next/theme-next-needmoreshare2@1/needsharebutton.min.js
    css: //cdn.jsdelivr.net/gh/theme-next/theme-next-needmoreshare2@1/needsharebutton.min.css
  postbottom:
    enable: true
    options:
      iconStyle: box
      boxForm: horizontal
      position: bottomCenter
      networks: Weibo,Wechat,Douban,QQZone,Twitter,Facebook
  float:
    enable: false
    options:
      iconStyle: box
      boxForm: horizontal
      position: middleRight
      networks: Weibo,Wechat,Douban,QQZone,Twitter,Facebook
```

## 打开收缩功能

旧版 直接修改配置即可

```
auto_excerpt:
  enable: true
  length: 150
```

> 就是MD格式没了，特别乱

新版

方法1：最简单的方式是用它默认的，改改配置文件即可

```
excerpt_description: true
read_more_btn: true
```

在文章的头部中的desc标签中：添加信息

或在内容中添加标签：

```
<!-- more -->
```

方法2：得安插插件 ：

> npm install hexo\-excerpt \-\-save

修改配置信息：

```
excerpt:
  depth: 5  
  excerpt_excludes: []
  more_excludes: []
  hideWholePostExcerpts: true
```

> 推荐方法2，主要改一个地方全文生效，省去麻烦

## 彩色标签

文章详情，尾部的标签加上icon图标\(默认是个\#号\)

tag\_icon: true

tag页面增加彩色：

> https://blog.csdn.net/qq\_39974578/article/details/114172260

创建一个配色文件：

> touch /data/www/golang/zblog/themes/next/layout/tag\-color.swig

添加配色代码：

> vim /data/www/golang/zblog/themes/next/layout/tag\-color.swig

vim themes/next/layout/page.swig

```
{% include 'tag-color.swig' %}
```

侧边栏增加tags：

npm install hexo\-tag\-cloud@^2.1.\* \-\-save

vim /data/www/golang/zblog/themes/next/layout/\_macro/sidebar.swig

```
{% if site.tags.length > 1 %}
<script type="text/javascript" charset="utf-8" src="{{ url_for('/js/tagcloud.js') }}"></script>
<script type="text/javascript" charset="utf-8" src="{{ url_for('/js/tagcanvas.js') }}"></script>
<div >
    <h3 >Tag Cloud</h3>
    <div >
        <canvas width="250" height="250" style="width:100%">
            {{ list_tags() }}
        </canvas>
    </div>
</div>
{% endif %}
```

```
# hexo-tag-cloud
tag_cloud:
    textFont: Trebuchet MS, Helvetica
    textColor: '#333'
    textHeight: 25
    outlineColor: '#E2E1D1'
    maxSpeed: 0.1
```

给文章底部tag 添加彩色:

vim /data/www/golang/zblog/themes/next/layout/\_macro/post.swig

```
 <div >
            {%- for tag in post.tags.toArray() %}
              <a   rel="tag"><i ></i> {{ tag.name }}</a>
            {%- endfor %}
          </div>
          <script type="text/javascript">
            var tagsall=document.getElementsByClassName("post-tags")
            for (var i = tagsall.length - 1; i >= 0; i--){
                var tags=tagsall[i].getElementsByTagName("a");
                for (var j = tags.length - 1; j >= 0; j--) {
                    var golden_ratio = 0.618033988749895;
                    var s = 0.5;
                    var v = 0.999;
                    var h = golden_ratio + Math.random()*0.8 - 0.5;
                    var h_i = parseInt(h * 6);
                    var f = h * 6 - h_i;
                    var p = v * (1 - s);
                    var q = v * (1 - f * s);
                    var t = v * (1 - (1 - f) * s);
                    var r, g, b;
                    switch (h_i) {
                        case 0:
                            r = v;
                            g = t;
                            b = p;
                            break;
                        case 1:
                            r = q;
                            g = v;
                            b = p;
                            break;
                        case 2:
                            r = p;
                            g = v;
                            b = t;
                            break;
                        case 3 :
                            r = p;
                            g = q;
                            b = v;
                            break;
                        case 4:
                            r = t;
                            g = p;
                            b = v;
                            break;
                        case 5:
                            r = v;
                            g = p;
                            b = q;
                            break;
                        default:
                            r = 1;
                            g = 1;
                            b = 1;
                      }
                    tags[j].style.background = "rgba("+parseInt(r*255)+","+parseInt(g*255)+","+parseInt(b*255)+","+0.5+")";
                }
            }                        
            </script>

```

```
<style>

.posts-expand .post-tags a {
    display: inline-block;
    font-size: 0.8em;
    padding: 0px 10px;
    border-radius: 8px;
    color: rgb(85, 85, 85);
    border: 0px;
}

</style>
```

vim /data/www/golang/zblog/themes/next/\_config.yml

```
<span >
    ${iconText('far fa-comment', 'valine','评论数')}
    <a title="valine"   itemprop="discussionUrl">
      <span data-xitemprop="commentCount"></span>
    </a>
  </span>
```

## 侧边栏添加分隔线

/Users/mayanyan/data/www/golang/zblog/themes/next/source/css/\_common/components/post/post\-eof.styl

```
.sidebar-eof {
  background: $grey-light;
  
  text-align: center;
  width: 100%;
}
```

## 浏览进度

scrollpercent: true

## 评论

先去LeanCloud注册个账号，再创建个应用，拿到appId appKey，设置到配置文件中

valine

文章详情页：标题下面的评论数非中文，有点BUG，改一下

vim themes/next/scripts/filters/comment/valine.js

## 添加图片列表功能

创建一个文件：/data/www/golang/zblog/themes/next/scripts/show\_some.js

```
var fs = require("fs");

hexo.extend.tag.register('show_some', function(args){

    if(!args[0]){
	return false;
    }

    var listStr = "";
    const files = fs.readdirSync(args[0]);
    const prefix = args[1];

    files.forEach(function (item, index) {
        var thisPicLink = prefix+item;
        var divStyle = "float:left";
        var oneRow = "<div  style='margin-right:10px;"+divStyle+"'><div>"+item+"</div>"

        oneRow += "<div><a href='"+thisPicLink+"' target='_blank' >"  +"<img src='" + thisPicLink + "' width=200 height=400 > </a> </div>";
        //display: inline-block
        oneRow += "</div>";
        listStr += oneRow;
    })

  listStr += "<div style='clear: left'></div>";

    return listStr ;
});

```

图片排队样式

vim \_common/scaffolding/tags/group\-pictures.styl

```
.page-post-detail .post-body .group-picture-column {
  // float: none;
  margin-top: 10px;
  // width: auto !important;
  img { margin: 0 auto; }
}

```

最后哪里使用，哪里引用 ：

> {% show\_some /data/www/golang/zblog/source/images/脑图/ /images/脑图/ %}

## 快速重启

hexo clean && hexo g && hexo s

## 尾部设置

```
footer:
  # Specify the date when the site was setup. If not defined, current year will be used.
  since: 2022

  # Icon between year and copyright info.
  icon:
    # Icon name in Font Awesome. See: https://fontawesome.com/icons
    #name: fa fa-heart
    name: fa fa-bug
    # If you want to animate the icon, set it to true.
    animated: true
    # Change the color of icon, using Hex Code.
    color: "#ff0000"
    #color: "#000000"
```

## 收尾，加入git仓

发现next主题是带着git版本的，不能直接把全部代码加入到git里

git status 看看是什么情况？

发现：除了配置文件，还有一个：头像图片有变动

先把配置文件备份下，再备份下图片

现在

git上创建一个项目,clone 一下代码：

> cd /data/www/golang
> 
> 
> git clone git@github.com:mqzhifu/zblog.git

## 总结

虽然说是hexo博客，但是对hexo的配置很少，大部分都是在调整next主题。而修改next主题就是修改页面
