FROM php:7.4-cli

# Dependent installation
RUN apt-get update

RUN apt install -y --no-install-recommends procps coreutils cron

# php default ext
RUN docker-php-ext-install  -j$(nproc) bcmath calendar exif gettext dba mysqli pcntl pdo_mysql shmop sysvmsg sysvsem sysvshm

# php zip ext
RUN apt-get install -y --no-install-recommends zip unzip libzip-dev
RUN docker-php-ext-install zip

# php gd ext
RUN apt-get install -y --no-install-recommends libfreetype6-dev libjpeg62-turbo-dev libpng-dev
RUN docker-php-source extract && cd /usr/src/php/ext/gd \
    && docker-php-ext-configure gd --with-jpeg=/usr/include --with-freetype=/usr/include/freetype2 \
    && make && make install \
    && docker-php-ext-install gd

# http://pecl.php.net/package/redis
ARG REDIS_VERSION=5.3.6
# php redis ext
RUN pecl install http://pecl.php.net/get/redis-${REDIS_VERSION}.tgz && docker-php-ext-enable redis

# php mongodb ext
# http://pecl.php.net/package/mongodb
ARG MONGODB_VERSION=1.11.1
RUN pecl install http://pecl.php.net/get/mongodb-${MONGODB_VERSION}.tgz && docker-php-ext-enable mongodb

# http://pecl.php.net/package/rdkafka
ARG RDKAFKA_VERSION=4.1.2
RUN apt-get install -y --no-install-recommends autoconf make gcc wget
# php rdkafka ext
RUN apt install -y --no-install-recommends librdkafka-dev
RUN pecl install http://pecl.php.net/get/rdkafka-${RDKAFKA_VERSION}.tgz && docker-php-ext-enable rdkafka


# http://pecl.php.net/package/swoole
ARG SWOOLE_VERSION=4.8.8
RUN docker-php-ext-install  -j$(nproc) sockets
# # php swoole ext
RUN apt-get install -y --no-install-recommends openssl libssl-dev libcurl4-openssl-dev
RUN cd /tmp && wget https://github.com/swoole/swoole-src/archive/v${SWOOLE_VERSION}.tar.gz -O swoole.tar.gz \
    && mkdir /tmp/swoole \
    && tar -xf swoole.tar.gz -C /tmp/swoole --strip-components=1 \
    && cd /tmp/swoole && phpize \
    && ./configure --enable-openssl --enable-http2 --enable-swoole-json --enable-swoole-curl --enable-mysqlnd --enable-sockets \
    && make && make install \
    && docker-php-ext-enable swoole

# http://pecl.php.net/package/solr
ARG SOLR_VERSION=2.5.0
# php solr ext
RUN apt-get install -y --no-install-recommends libcurl4-gnutls-dev libxml2-dev
RUN pecl install http://pecl.php.net/get/solr-${SOLR_VERSION}.tgz && docker-php-ext-enable solr


RUN rm -rf /tmp/*
RUN rm -rf /var/log/*
RUN rm -rf /usr/src/php && rm -rf /var/lib/apt/lists/*

