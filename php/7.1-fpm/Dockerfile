FROM php:7.1-fpm

MAINTAINER Konstantin Voloshchuk <kosv.dev@gmail.com>

ARG HOST_USER_UID=1000
ARG HOST_USER_GID=1000

ARG PHP_WORK_DIR="/var/www"

ARG XDEBUG_REMOTE_CONNECT_BACK=0
ARG XDEBUG_REMOTE_PORT=9001
ARG XDEBUG_REMOTE_IDEKEY="PHPSTORM"


RUN set -ex                                                 && \
                                                               \
    echo 'Creating notroot user and group from host'        && \
    groupadd -g $HOST_USER_GID notroot                      && \
    useradd -lm -u $HOST_USER_UID -g $HOST_USER_GID notroot

RUN apt-get update && apt-get install -y \
        curl \
        wget \
        git \
        libfreetype6-dev \
        libjpeg62-turbo-dev \
        libmcrypt-dev \
        libpng12-dev

RUN docker-php-ext-install -j$(nproc) iconv mcrypt mbstring mysqli pdo pdo_mysql zip \
        && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
        && docker-php-ext-install -j$(nproc) gd

# XDebug
RUN yes | pecl install xdebug \
    && chmod +x $(find /usr/local/lib/php/extensions/ -name xdebug.so) \
    && echo "zend_extension=$(find /usr/local/lib/php/extensions/ -name xdebug.so)" > /usr/local/etc/php/conf.d/xdebug.ini \
    && echo "xdebug.remote_enable=on" >> /usr/local/etc/php/conf.d/xdebug.ini \
    && echo "xdebug.remote_autostart=on" >> /usr/local/etc/php/conf.d/xdebug.ini \
    && echo "xdebug.remote_handler=\"dbgp\"" >> /usr/local/etc/php/conf.d/xdebug.ini \
    && echo "xdebug.remote_connect_back=$XDEBUG_REMOTE_CONNECT_BACK" >> /usr/local/etc/php/conf.d/xdebug.ini \
    && echo "xdebug.remote_port=$XDEBUG_REMOTE_PORT" >> /usr/local/etc/php/conf.d/xdebug.ini \
    && echo "xdebug.remote_mode=req" >> /usr/local/etc/php/conf.d/xdebug.ini \
    && echo "xdebug.idekey=\"$XDEBUG_REMOTE_IDEKEY\"" >> /usr/local/etc/php/conf.d/xdebug.ini

# Composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# Codeception
RUN curl -LsS http://codeception.com/codecept.phar -o /usr/local/bin/codecept \
    && chmod a+x /usr/local/bin/codecept

ADD php.ini /usr/local/etc/php/conf.d/40-custom.ini

WORKDIR $PHP_WORK_DIR

CMD ["php-fpm"]