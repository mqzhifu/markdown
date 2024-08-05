# 基础使用

第一次安装/初始化，配置好 composer.json
>composer install

更新所有包
>composer.phar update

安装指定包，指定版本
>composer.phar require rouchi/jy\_proxy\_client ~v1.3.4

查找包
>composer search

列出可用包
>composer show

设置composer
>composer config XXXXXX

参数 ：-g 全局

查看 composer 配置信息
>composer config \-\-list

清理缓存
>composer clear-cache

# 更改源

composer config repo.packagist composer https://mirrors.aliyun.com/composer/
composer config repo.packagist composer http://packagist.rouchi.com/

https://packagist.phpcomposer.com
https://mirrors.aliyun.com/composer/
https://packagist.laravel\-china.org
https://pkg.phpcomposer.com

腾讯
https://mirrors.cloud.tencent.com/composer

官方
https://packagist.org/

也可以使用配置文件

```
"repositories": {
	"packagist": {
	"type": "composer",
	"url": "https://packagist.rouchi.com"
	}
},
```

不使用https

composer \-g config secure\-http false

也可以使用配置文件
```
"config": {
	"secure\-http": false
}

```

composer config \-g cache\-dir "D:\\Program Files \(x86\)\\composer\\cache"
composer config \-g data\-dir "D:\\Program Files \(x86\)\\composer\\data"
composer config \-g cache\-files\-dir 'D:\\Program Files \(x86\)\\composer\\cache\-files'

