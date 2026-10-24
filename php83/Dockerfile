FROM php:8.3.36-fpm-alpine

LABEL maintainer="ogenes.yi@gmail.com"

ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# 换阿里云源
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories

# 系统依赖（linux-headers 是关键）
RUN apk add --no-cache \
    curl git unzip zip sudo \
    linux-headers \
    icu-dev libzip-dev oniguruma-dev \
    libjpeg-turbo-dev libpng-dev freetype-dev \
    openssl-dev libxml2-dev bzip2-dev \
    librdkafka-dev libmemcached-dev

# PHP 核心扩展
RUN docker-php-ext-configure gd --with-jpeg --with-freetype \
 && docker-php-ext-install -j$(nproc) \
    bcmath intl mbstring mysqli pcntl pdo_mysql \
    sockets zip soap exif gd opcache

# PECL 扩展
ARG EXT_REDIS=false
RUN if [ "$EXT_REDIS" = "true" ]; then \
    apk add --no-cache $PHPIZE_DEPS \
    && pecl install -o -f redis \
    && docker-php-ext-enable redis \
    && apk del $PHPIZE_DEPS; \
    fi

ARG EXT_SWOOLE=false
RUN if [ "$EXT_SWOOLE" = "true" ]; then \
    apk add --no-cache $PHPIZE_DEPS \
    && pecl install -o -f swoole \
    && docker-php-ext-enable swoole \
    && apk del $PHPIZE_DEPS; \
    fi

# Composer
COPY --from=composer:2 /usr/bin/composer /usr/bin/composer

ENV COMPOSER_HOME /var/www/.composer
RUN mkdir -p $COMPOSER_HOME/cache && chown -R www-data:www-data $COMPOSER_HOME

RUN chown -R www-data:www-data /var/www/
RUN mkdir -p /var/log/php && chown -R www-data:www-data /var/log/php
RUN mkdir -p /data/log && chown -R www-data:www-data /data/log

VOLUME $COMPOSER_HOME
WORKDIR /var/www
