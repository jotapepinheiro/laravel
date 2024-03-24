ARG PHP_VERSION
FROM php:${PHP_VERSION}

## Diretório da aplicação
ARG APP_DIR=/var/www/html

## Diretório dos arquivos de configuração
ARG DOCKER_DIR=./docker/build

## Versão da Lib do Redis para PHP
ARG REDIS_LIB_VERSION=5.3.7

### apt-utils é um extensão de recursos do gerenciador de pacotes APT
RUN apt-get update -y && apt-get install -y --no-install-recommends \
    apt-utils \
    supervisor

# Dependências recomendadas de desenvolvido para ambiente linux
RUN apt-get update && apt-get install -y \
    zlib1g-dev \
    libzip-dev \
    unzip \
    libpng-dev \
    libpq-dev \
    libxml2-dev

RUN docker-php-ext-install mysqli pdo pdo_mysql pdo_pgsql pgsql session xml

# Redis
RUN pecl install redis-${REDIS_LIB_VERSION} \
    && docker-php-ext-enable redis

RUN docker-php-ext-install zip iconv simplexml pcntl gd fileinfo

# Composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

### Supervisor
COPY ${DOCKER_DIR}/supervisord/supervisord.conf /etc/supervisor/conf.d/supervisord.conf
#COPY ${DOCKER_DIR}/supervisord/conf /etc/supervisord.d/

COPY ${DOCKER_DIR}/php/extra-php.ini "$PHP_INI_DIR/99_extra.ini"
COPY ${DOCKER_DIR}/php/extra-php-fpm.conf /etc/php8/php-fpm.d/www.conf

WORKDIR $APP_DIR
RUN cd $APP_DIR
RUN chown www-data:www-data $APP_DIR

COPY --chown=www-data:www-data ./html .
RUN rm -rf vendor
RUN composer install --no-interaction

### NGINX
RUN apt-get install nginx -y
RUN rm -rf /etc/nginx/sites-enabled/* && rm -rf /etc/nginx/sites-available/*
COPY ${DOCKER_DIR}/nginx/sites.conf /etc/nginx/sites-enabled/default.conf
COPY ${DOCKER_DIR}/nginx/error.html /var/www/html/error.html

RUN apt-get clean && rm -rf /var/lib/apt/lists/*
# RUN apt update -y && apt install nano git -y

CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/supervisord.conf"]
