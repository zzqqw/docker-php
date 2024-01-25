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


docker build --build-arg="IMAGE_TAG=7.4-fpm" -f Dockerfile.nginx -t zhiqiangwang/php:7.4-fpm-nginx  .
~~~


## 启动容器
~~~
docker run --name myphp -p 9000:9000 -v /data/wwwroot/laravel/:/var/www/html -d zhiqiangwang/php:7.4-fpm 

docker run --name myphp -p 8080:80 -d zhiqiangwang/php:7.4-fpm-nginx
~~~
