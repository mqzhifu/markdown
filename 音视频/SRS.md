# SRS

官网

> https://ossrs.net

## 安装

下载代码

> git clone \-b 4.0release https://gitee.com/ossrs/srs.git

编译

```
cd srs/trunk
./configure
make
```

启动：

> ./objs/srs \-c conf/srs.conf

检查：

> http://localhost:8080/

查看SRS的状态

> ./etc/init.d/srs status

者看SRS的日志

> tail \-n 30 \-f ./objs/srs.log

停止

> ./etc/init.d/srs stop

大概看一下，主配置文件中端口号，vim conf/srs.conf:

|端口号|协议      |描述                  |备注                                                                    |
|------|----------|----------------------|------------------------------------------------------------------------|
|1935  |RMTP      |推流                  |                                                                        |
|8080  |HTTP      |一个可视化网页管理后台|                                                                        |
|1985  |HTTP\-api |没搞懂                |没太理解为毛又分拆了一个端口，完全可以共用8080。我猜可能是后端分拆了模块|
|8088  |https     |同8080一样            |webRtc浏览器测试必须得是HTTPS                                           |
|1990  |HTTPS\-api|同1985一样            |webRtc浏览器测试必须得是HTTPS                                           |

测试一下用 ffmpeg 本地推流：

## 测试直播

前置条件：

1. 准备一个视频文件，如果是 FLV 比较好，MP4的话得转一下 FLV 格式

> /soft/ffmpeg/bin/ffmpeg \-i test.mp4 \-c copy test.flv

1. ffmpeg

> 服务器先安装 ffmpeg，centos/mac 下一键搞定，ubutu 很麻烦，源码编译安装....

#### 开始推流

> /soft/ffmpeg/bin/ffmpeg \-re \-i /install/test.mp4 \-c copy \-f flv rtmp://localhost/live/livestream

#### 拉流

1. H5\(HLS\)，这个最简单浏览器直接打开地址即可\(浏览器可能还得安装插件\)
    > http://8.142.177.235:8080/live/livestream.m3u8
2. H5\(HTTP\-FLV\)，这个略麻烦点分成两种情况吧：客户端和浏览器
    1. ffmpeg ，这个直接可以用
    > ffplay \-i "http://8.142.177.235:8080/live/livestream.flv" \-fflags nobuffer
    1. 写点JS代码，用flv.js库，最后填写服务器 URL 地址即可：
    > https://github.com/bilibili/flv.js
    > 
    > 
    > http://8.142.177.235:8080/live/livestream.flv
    > 
    > 
    > 早期是浏览器安装 flash\_player 插件，可以直接播放。之后flash 被放弃，浏览器还有些其它插件播放，后期基本上这些3方插件也被谢谢了，只有用JS库。
3. RTMP: rtmp://8.142.177.235//live/livestream

> VLC是个客户端软件\(播放器\)，去官网下一个，然后打开，里面配置一个这个URL地址即可。

## webrtc

这个略恶心人，浏览器 强制要求必须得HTTPS，所以它就得单搞：

修改配置文件：https.rtc.conf

```
candidate 8.142.177.235;
listen 8000;#UDP 

key /data/www/srs/9245472_webrtc-push.seedreality.com.key;
cert /data/www/srs/9245472_webrtc-push.seedreality.com.pem;

```

启动：\(这里换了个配置文件\)

> ./objs/srs \-c conf/https.rtc.conf

测试\-浏览器推流：

> https://webrtc\-push.seedreality.com:8088/players/rtc\_publisher.html?autostart=true&stream=livestream&api=1990&schema=https

测试\-浏览器播放：

?\>https://webrtc\-push.seedreality.com:8088/players/rtc\_player.html?autostart=true&stream=livestream&api=1990&schema=https

## ubutu安装ffmpeg

下载地址：

> https://ffmpeg.org/download.html
> 
> 
> https://launchpad.net/ubuntu/\+archive/primary/\+sourcefiles/ffmpeg/7:5.1.1\-1ubuntu1/ffmpeg\_5.1.1.orig.tar.xz

```
xz -dk ffmpeg_5.1.1.orig.tar.xz
tar xvf ffmpeg_5.1.1.orig.tar
cd ffmpeg-5.1.1/
```

```
安装ffplay需要的依赖
sudo apt-get install libx11-dev xorg-dev libsdl2-2.0 libsdl2-dev
sudo apt install yasm pkg-config libopencore-amrnb-dev libopencore-amrwb-dev
```

mkdir \-p /soft/ffmpeg

./configure \-\-prefix=/soft/ffmpeg \-\-enable\-gpl \-\-enable\-version3 \-\-enable\-nonfree \-\-enable\-ffplay \-\-enable\-ffprobe \-\-enable\-debug

下面3个找不到，先不管了

\-\-enable\-libfdk\-aac \-\-enable\-libx264 \-\-enable\-libx265
