letsencrypt 一个免费生成HTTPS 证书的软件

官网：
https://certbot.eff.org/

==========这种方式已经不支持了===========

wget https://dl.eff.org/certbot-auto

chmod a+x ./certbot-auto

./certbot-auto --help

sudo mv certbot-auto /usr/local/bin/

sudo chown root /usr/local/bin/certbot-auto

sudo chmod 0755 /usr/local/bin/certbot-auto

==================================

使用官网的方式吧

centos apache

首先提示安装 ：Install snapd

坑B，发现得升级 centos linux 内核版本，这不扯呢嘛

//续签

./certbot renew

//续签-强制

./certbot renew --force-renew

//注销证书

certbot revoke --cert-path /etc/letsencrypt/live/api.day900.com/cert.pem

// 删除证书（撤销之后使用）

certbot delete --cert-name example.com

//操的，整整折腾一圈，坑死人。最后将centos 回退 由7.9改到7.5

sudo yum install epel-release

sudo yum install snapd

sudo systemctl enable --now snapd.socket

sudo ln -s /var/lib/snapd/snap /snap

sudo snap install core; sudo snap refresh core

sudo snap install --classic certbot

sudo ln -s /snap/bin/certbot /usr/bin/certbot

certbot certonly --manual -d *.xlsyfx.cn --agree-tos --preferred-challenges dns-01 --server https://acme-v02.api.letsencrypt.org/directory

certonly 表示只申请证书。

--agree-tos 同意ACME协议。

--no-bootstrap 需要用户同意的系统级操作直接选N。

--manual-public-ip-logging-ok 自动允许ip被记录，默认是询问，如果不同意将不能申请通过。

--manual 表示交互式申请。

-d 为那些主机申请证书如 *.xxx.cn（此处为泛域名）

--preferred-challenges dns，使用 DNS 方式校验域名所有权，可以配置多个

--server Let's Encrypt ACME v2 版本使用的服务器不同于 v1 版本，需要显示指定。

NND，他这个，为了验证该域名是否属于本人，要发起挑战，需要你在域名解析的过程中添加一个txt记录，没添加这个记录，被官方给限流了。。。

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

Please deploy a DNS TXT record under the name

_acme-challenge.xlsyfx.cn with the following value:

nLQ0QcaQQHmupnlJJcEuW_f7c2xyLlYgRpFn4ZTwasM

Before continuing, verify the record is deployed.

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

去域名管理里，添加一个txt 记录，把字符串加进去，等个10分钟