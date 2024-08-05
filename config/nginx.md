# nginx

## nginx.conf

```
#进程启动的用户名
user www;
#进程ID存在路径
pid logs/nginx.pid;

#woker 进程数量。建议与CPU核数对应，理想状态下：WORK 进程会分散在各个CPU上
worker_processes auto;
#WORK进程绑定CPU，避免 进程被切换到其它CPU上，更新CPU缓存
worker_cpu_affinity auto;
#错误日志位置
error_log /data/logs/nginx/error.log notice;

event{
    use epoll;
    worker_connections 1024;#每个work最大
    multi_accept on;#设置一个进程是否同时接受多个网络连接，默认为off
    accept_mutex on; #避免惊群,默认打开
}

http {
    #隐藏版本号，正常HTTP交互，header中Server值会显示nginx/18.0，还有404会直接把版本号显示在页面中
    server_tokens off;

    #标签：main ，缓存32K内存后输出到硬盘，每5秒flush一次，注：得配置log_format一起使用，不然报错
    access_log /data/logs/nginx/access.log main gzip buffer=32k flush=5s;

    #nginx读取本地文件，需要用户模式/系统模式切换，开启后，直接让CPU来读文件，0-copy
    sendfile on;
    #数据包会累计到一定大小之后才会发送，减小了额外开销(开启sendfile此参数才生效)
    tcp_nopush on;
    #禁用 Nagle 算法，只对keep-alive的TCP 连接有用，好像与tcp_nopush互斥
    tcp_delay on;
    #长连接超时时间
    keepalived_timeout 60;

    #C端最大上传文件
    client_max_body_size 2m;
    #C端上传的文件，小于如下值会保存在内存中，如果大于此值会放于文件
    client_body_buffer_size 1m;

    #缓存(传输数据)设置
    gzip on;
    gzip_comp_level 5; # 1-9
    gzip_min_length 100k;
    gzip_types application/javascript text/css text/xml;
    gzip_vary on; #告诉客户端开启了压缩

    #fastcgi php
    fastcgi_connect_timeout 60;#cgi 连接超时60s
    #fastcgi_send_timeout;#向cgi发送时间
    fastcgi_read_timeout 60;#接收php响应时间

    #申请一块区域10M，存储$binary_remote_addr(ip)变量的统计(限制IP)
    limit_conn_zone $binary_remote_addr zone=addr:10m;
    
    #申请一块区域10M，存储$binary_remote_addr(domain)变量的统计
    limit_conn_zone $server_name zone=perserver:10m;
    
    #申请一块区域10M，存储$binary_remote_addr(ip)变量的统计，每秒只能有一个请求进来(限制一个IP的请求数)
    limit_req_zone $binary_remote_addr zone=mylimit:10m rate=1r/s;
    
    location / {
        #一个IP同一时间最多一个连接,addr 对应上面的 addr
        limit_conn addr 1;
        #每个连接限速300K,是针对连接的，如果一个IP最多可以有2个连接，那就是每个连接300K
        limit_rate 300k;
        #设置超出最大连接数的日志级别
        limit_conn_log_level error;
        #当有请求过来触发了最大请求数，由burst缓存住5个请求
        #nodelay：当请求频次 被限制burst缓存也满了，直接扔掉,return 503
        limit_req zone=mylimit burst=5 nodelay;
    }
    
    include vhost/*.conf;
    

}

```

## metrics

```

vhost_traffic_status_zone;
server {
    listen 9111;
    location /status {
    vhost_traffic_status_display;
    vhost_traffic_status_display_format html;
    }
}
```

#### server

```
server {
    autoindex off;
    access_log /data/logs/nginx/agent.xlsyfx.cn.access.log;
    server_name agent.xlsyfx.cn;

    listen 80;
    listen 443 ssl;
    ssl_certificate /etc/letsencrypt/live/xlsyfx.cn/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/xlsyfx.cn/privkey.pem;

    ssl_session_timeout 5m;
    ssl_protocols SSLv3 TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers "HIGH:!aNULL:!MD5 or HIGH:!aNULL:!MD5:!3DES";
    ssl_prefer_server_ciphers on;

    error_page 404 /404.html;
        location = /404.html {
        root /data/www;
    }

    error_page 500 502 503 504 /50x.html;
        location = /50x.html {
        root /data/www;
    }

    root /data/www;
        location / {
        index index.php index.html;
    }
    
    #图片映射本机硬盘地址
    location ~* \.(gif|jpg|jpeg|png|css|js|ico)$ {
        root /data/www/php/zhongyuhuacai/static;
    }
    #特殊文件映射本机硬盘地址
    location ~* \.(eot|otf|ttf|woff|woff2)$ {
        root /data/www/php/zhongyuhuacai/static;
        add_header Access-Control-Allow-Origin *;
    }

    #对URI进行转换
    location ~ / {
        if (!-e $request_filename) {
            rewrite ^/(\w+)/(\w+)/(.*)$ /index.php?ctrl=$1&ac=$2&$3 last;
        }
    }

    #解析PHP
    location ~ \.php$ {
        root /data/www;
        fastcgi_pass 127.0.0.1:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        #fastcgi_param SCRIPT_FILENAME /data/www/php/zhongyuhuacai/api$fastcgi_script_name;
        include fastcgi_params;
        fastcgi_param HTTP_X_REAL_IP $HTTP_X_REAL_IP;
    }
}

```

## 代理/负载均衡

```
upstream my_server {
    server 127.0.0.1:8080 weight=5;
    server 10.0.0.2:8080;
    keepalive 2000;
}
server {
    listen 80;
    server_name 10.0.0.1;
    location /my/ {
    proxy_pass http://my_server/;
    proxy_set_header Host $host:$server_port;
    }
}
```

## trace\_id

```
server {  
        lua_code_cache off;
        location ~ \.php$ {
            set $first 0;
            if ( $http_x_trace_){
                set $first 1;
            }

            if ( $http_x_trace_id = 0){
                set $first 1;
            }

            if ( $http_x_trace_){
                set $first 1;
            }

            set_by_lua $span_id 'return "200000"..os.time()';

            if ( $first = 1 ){
                set_by_lua_file $trace_id /home/www/test_lua/trace_id.lua;
                set $first 1;
            }

            if ( $first = 0 ){
                set $trace_id $http_x_trace_id;
                set $parent_id $http_x_span_id;
            }

            more_set_input_headers "X-Parent-Id:$parent_id";
            more_set_input_headers "X-Trace-Id:$trace_id";
            more_set_input_headers "X-Span-Id:$span_id";
            more_set_input_headers "X-From-Service:$server_name";
            more_set_input_headers "X-First:$first";
        
        }

}
```

## VUE

```
server {

    location / {
        root /data/www/Digitaltwin/master/dist;
        try_files $uri $uri/ @router;
        index index.html;
    }

    location @router {
        rewrite ^.*$ /index.html last;
    }
}
```

## golang

```
server {
    location / {
        proxy_set_header  X-Real-IP $remote_addr;
        proxy_set_header  REMOTE-HOST $remote_addr;
        proxy_set_header  X-Forwarded-For $proxy_add_x_forwarded_for;

    
        proxy_pass http://127.0.0.1:5555;

        proxy_connect_timeout 30;
        proxy_send_timeout 300;
        proxy_read_timeout 300;
    }
}
```
