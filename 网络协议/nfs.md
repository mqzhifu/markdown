Network File System

# ubuntu安装

sudo apt\-get install nfs\-kernel\-server \# 安装 NFS服务器端
sudo apt\-get install nfs\-common \# 安装 NFS客户端
sudo /etc/init.d/nfs\-kernel\-server start
sudo /etc/init.d/nfs\-kernel\-server restart

exportfs \-arv

tcp:111 2049

udp:111 4046

showmount \-e 8.142.177.235

showmount.exe \-e 8.142.177.235

nfs.client.mount.options = vers=4

/data/nfs \*\(rw,sync,no\_root\_squash,no\_subtree\_check,insecure\)

mount \-t nfs \-o resvport 8.142.177.235:/data/nfs /Users/mayanyan/data/test\_nfs

win10

打开或关闭Windows功能选项\-\> NFS服务

## smaba

修改配置文件：

```
[global]
min protocol = NT1#小米电视只能支持1.0版本的samba协议

[share]
comment = share
path = media/mayanyan/软件/guoguo
public = yes
browseable = yes
writable = yes
read only = false
valid users = mayanyan              //用户名，ubuntu的user
directory mask = 0777
#force user = graysen 
#force group = graysen 
available = yes
guest ok = yes
```

#### 给硬盘修改 label

wxfat格式：（apt install exfatprogs）

> exfatlabel /dev/sdb2 dior

ntfs格式：

> ntfslabel /d

查看无线网站：

> wconfig

列出当前USB端口上的设备信息

> lsusb
