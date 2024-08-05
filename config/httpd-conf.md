# httpd.conf

```
liseten 80
User www
Group www

ErrorLog "/data/logs/apache/error_log"
CustomLog "/data/logs/apache/access_log" common

ServerName 39.106.65.76:80

DocumentRoot "/data/www"
DirectoryIndex index.htm index.html index.php
AddType application/x-httpd-php .php .phtml .php3 .inc 

#关闭掉apache目录显示
Options Indexes FollowSymLinks

#允许rewrite .ht文件
rewrite_module modules/mod_rewrite.so

AllowOverride Index #这行代码会有问题
AllowOverride All

Include conf/extra/httpd-info.conf

支付特殊字体类型请求

<FilesMatch "\.(ttf|otf|eot|woff|svg|woff2)$">
  <IfModule mod_headers.c>
    Header set Access-Control-Allow-Origin "*"
  </IfModule>
</FilesMatch>

<VirtualHost *:80>
ServerName majiang.com
DocumentRoot D:\www\z
</VirtualHost>
```

> 

## https

> 打开注释
> 
> 
> LoadModule ssl\_module modules/mod\_ssl.so
> 
> 
> LoadModule socache\_shmcb\_module modules/mod\_socache\_shmcb.so
> 
> 
> Include conf/extra/httpd\-ssl.conf

cd /soft/apache/conf/extra/

vim httpd\-ssl.conf

SSLCertificateFile "/etc/letsencrypt/live/xlsyfx.cn/fullchain.pem"

SSLCertificateKeyFile "/etc/letsencrypt/live/xlsyfx.cn/privkey.pem"
