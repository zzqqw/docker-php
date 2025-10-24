## 构建镜像

支持多个 PHP 版本和模式（CLI/FPM）：

~~~
# PHP 7.x
docker build -f 7.1-cli/Dockerfile -t zhiqiangwang/php:7.1-cli .
docker build -f 7.1-fpm/Dockerfile -t zhiqiangwang/php:7.1-fpm .

docker build -f 7.2-cli/Dockerfile -t zhiqiangwang/php:7.2-cli .
docker build -f 7.2-fpm/Dockerfile -t zhiqiangwang/php:7.2-fpm .

docker build -f 7.3-cli/Dockerfile -t zhiqiangwang/php:7.3-cli .
docker build -f 7.3-fpm/Dockerfile -t zhiqiangwang/php:7.3-fpm .

docker build -f 7.4-cli/Dockerfile -t zhiqiangwang/php:7.4-cli .
docker build -f 7.4-fpm/Dockerfile -t zhiqiangwang/php:7.4-fpm .

# PHP 8.x
docker build -f 8.1-cli/Dockerfile -t zhiqiangwang/php:8.1-cli .
docker build -f 8.1-fpm/Dockerfile -t zhiqiangwang/php:8.1-fpm .

docker build -f 8.2-cli/Dockerfile -t zhiqiangwang/php:8.2-cli .
docker build -f 8.2-fpm/Dockerfile -t zhiqiangwang/php:8.2-fpm .
~~~

## 扩展镜像

```
# composer 支持
docker build --build-arg="IMAGE_TAG=7.4-fpm" -f Dockerfile.composer -t zhiqiangwang/php:7.4-fpm-composer .

# nginx + fpm 一体化
docker build --build-arg="IMAGE_TAG=7.4-fpm" -f Dockerfile.nginx -t zhiqiangwang/php:7.4-fpm-nginx .

# phpMyAdmin 镜像
docker build -f Dockerfile.phpmyadmin -t zhiqiangwang/php:phpmyadmin .
```

## 特殊镜像

**企业微信会话内容存档（wxwork-finance）镜像**已单独维护，见仓库：
 https://github.com/chihqiang/docker-php-wxwork-finance

~~~
zhiqiangwang/php:7.4-fpm-wxwork-finance
zhiqiangwang/php:7.4-cli-wxwork-finance
~~~

## 启动容器示例

~~~
docker run --name myphp -p 9000:9000 -v $(pwd):/var/www/html -d zhiqiangwang/php:7.4-fpm
~~~

### phpMyAdmin（基于 nginx + fpm）

```
docker run --name phpmyadmin -p 8080:80 -e PMA_HOST=192.168.3.110 -d zhiqiangwang/php:7.4-fpm-nginx
```

### 项目部署（ThinkPHP 示例）

```
docker run --name myphp -p 8080:80 \
  -v $(pwd):/var/www/html \
  -v $(pwd)/docker/thinkphp.conf:/etc/nginx/sites-enabled/default \
  -d zhiqiangwang/php:7.4-fpm-nginx
```

## 常见配置目录

- `/etc/nginx`            — nginx 配置
- `/usr/local/etc`        — PHP/FPM 配置
- `/etc/supervisor`       — supervisor 配置
- `/var/spool/cron/crontabs` — crontab 任务计划

## 常见命令

~~~
# 添加定时任务
echo "* * * * * Command" >> /var/spool/cron/crontabs/root
chmod 600 /var/spool/cron/crontabs/root
/etc/init.d/cron start

# 启动方式
nginx -g "daemon off;"
php -i | grep php.ini
php-fpm --nodaemonize
~~~

## 修改上传文件限制

~~~

; 单个文件最大大小（设置为1GB，需带单位G）
upload_max_filesize = 1G
; 整个POST请求的最大大小（需略大于upload_max_filesize，避免表单其他数据占用空间）
post_max_size = 1.2G
; 脚本可使用的最大内存（需大于post_max_size，确保有足够内存处理请求）
memory_limit = 2G
; 脚本最大执行时间（1GB文件按普通带宽上传耗时较长，设为3小时=10800秒，可根据实际带宽调整）
max_execution_time = 10800
; 接收POST数据的最大时间（与执行时间保持一致，避免接收过程中超时）
max_input_time = 10800
; 允许同时上传的文件数量（默认10，若单请求传多个文件可增大，单文件上传保持默认即可）
max_file_uploads = 10


# 增加upload.ini
cat > /usr/local/etc/php/conf.d/upload.ini << EOF
upload_max_filesize = 1G
post_max_size = 1.2G
memory_limit = 2G
max_execution_time = 10800
max_input_time = 10800
max_file_uploads = 10        
EOF

# 修改nginx
; 允许的请求体最大大小（与PHP的upload_max_filesize一致，1GB）
client_max_body_size 1G;
; 延长连接超时时间（避免上传过程中连接被断开）
client_body_timeout 10800s;  # 接收请求体的超时时间
client_header_timeout 10800s;  # 接收请求头的超时时间
send_timeout 10800s;  # 发送响应的超时时间
keepalive_timeout 10800s;  # 长连接超时时间
~~~

## 日志打印到 Docker 控制台

```
ln -sf /dev/stdout /usr/local/openresty/nginx/logs/access.log
```

## 启动 Nginx + PHP 分离容器

1. #### 创建网络：

```
docker network create php-nginx-network
```

2. #### 启动 nginx：

```
docker run --name nginx -d -p 8080:80 -v tp:/code nginx:latest
```

3. #### 加入网络：

```
docker network connect php-nginx-network nginx
```

4. #### 启动 PHP 容器：

```
docker run --name myphp \
  -v tp:/var/www/html \
  --network php-nginx-network \
  -p 9000:9000 \
  -d zhiqiangwang/php:8.2-fpm-composer
```

🔍 **注意：**

- Nginx `root` 目录应设置为 PHP 容器中的 `/var/www/html`
- `fastcgi_pass` 配置为 `myphp:9000`
