# 构建

~~~
docker build -f Dockerfile.7.4-fpm -t zhiqiangwang/php:7.4-fpm  .
docker build -f Dockerfile.7.4-cli -t zhiqiangwang/php:7.4-cli  .

docker build -f Dockerfile.7.4-fpm-wxwork-finance -t zhiqiangwang/php:7.4-fpm-wxwork-finance .
docker build -f Dockerfile.7.4-cli-wxwork-finance -t zhiqiangwang/php:7.4-cli-wxwork-finance .

docker build -f Dockerfile.8.1-fpm -t zhiqiangwang/php:8.1-fpm  .
docker build -f Dockerfile.8.1-cli -t zhiqiangwang/php:8.1-cli  .

docker build -f Dockerfile.8.2-fpm -t zhiqiangwang/php:8.2-fpm  .
docker build -f Dockerfile.8.2-cli -t zhiqiangwang/php:8.2-cli  .
~~~


## 启动容器
~~~
docker run --name myphp -p 9000:9000 -v /data/wwwroot/laravel/:/var/www/html -d zhiqiangwang/php:7.4-fpm 
~~~

## 宿主nginx配置
~~~
server {
    listen 8080 default_server;

    # 容器目录
    root /var/www/html/laravel/public;

    # 静态资源
    location ~ .*\.(gif|jpg|jpeg|png|bmp|swf|css|js)$ {
        # 宿主目录
        root /data/wwwroot/laravel/public;
        if (-f $request_filename) {
            expires 1d;
            break;
        }
    }

    listen 443 ssl http2;
    ssl_certificate    /etc/nginx/ssl/www.zhiqiang.wang.cer;
    ssl_certificate_key   /etc/nginx/ssl/www.zhiqiang.wang.key;
    ssl_protocols TLSv1.1 TLSv1.2 TLSv1.3;
    ssl_ciphers EECDH+CHACHA20:EECDH+CHACHA20-draft:EECDH+AES128:RSA+AES128:EECDH+AES256:RSA+AES256:EECDH+3DES:RSA+3DES:!MD5;
    ssl_prefer_server_ciphers on;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-XSS-Protection "1; mode=block";
    add_header X-Content-Type-Options "nosniff";

    index index.html index.htm index.php;

    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    error_page 404 /index.php;

    large_client_header_buffers 4 16k;
    client_max_body_size 30m;
    client_body_buffer_size 128k;

    fastcgi_connect_timeout 300;
    fastcgi_read_timeout 300;
    fastcgi_send_timeout 300;
    fastcgi_buffer_size 64k;
    fastcgi_buffers   4 32k;
    fastcgi_busy_buffers_size 64k;
    fastcgi_temp_file_write_size 64k;

    # php-fpm
    location ~ \.php$ {
        fastcgi_pass 127.0.0.1:9000;
        fastcgi_index index.php;
        fastcgi_param    SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
~~~
