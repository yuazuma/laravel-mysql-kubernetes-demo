FROM node:12.13.0-alpine as node


FROM php:8.0.0-fpm-buster

COPY ./docker/app/php.ini /usr/local/etc/php/
COPY ./laravel /var/www/


# Composer
RUN apt-get update \
  && apt-get install -y zlib1g-dev libzip-dev mariadb-client \
  && docker-php-ext-install zip pdo_mysql

COPY --from=composer /usr/bin/composer /usr/bin/composer
ENV COMPOSER_ALLOW_SUPERUSER 1
ENV COMPOSER_HOME /composer
ENV PATH $PATH:/composer/vendor/bin


# Node.js
#npmのインストール
COPY --from=node:15.4 /usr/local/bin /usr/local/bin
COPY --from=node:15.4 /usr/local/lib /usr/local/lib


WORKDIR /var/www
RUN composer global require "laravel/installer"
RUN chmod 777 -R storage
