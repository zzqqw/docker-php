## 构建镜像

~~~
docker build -f php56-fpm -t zhiqiangwang/php:5.6-fpm .
docker build -f php:7.4-fpm -t zhiqiangwang/php:7.4-fpm  .

docker run --name myphp -v /data/wwwroot/laravel/:/var/www/html -d zhiqiangwang/php:7.4-fpm
~~~
