FROM php:7.2

LABEL maintainer="inhere <in.798@qq.com>" version="2.0"

######
# change sources.list
######
RUN mv /etc/apt/sources.list /etc/apt/sources.list.bak && \
echo \
deb https://mirrors.aliyun.com/debian buster main \
deb https://mirrors.aliyun.com/debian-security buster/updates main \
deb https://mirrors.aliyun.com/debian buster-updates main > /etc/apt/sources.list
######

# --build-arg timezone=Asia/Shanghai
ARG timezone
# app env: prod pre test dev
ARG app_env=local
# default use www-data user
# ARG work_user=www-data

ENV APP_ENV=${app_env:-"prod"} \
    TIMEZONE=${timezone:-"Asia/Shanghai"} \
    PHPREDIS_VERSION=5.3.1 \
    SWOOLE_VERSION=4.5.2 \
    COMPOSER_ALLOW_SUPERUSER=1 \
    XLSWRITER_VERSION=1.3.4.1

# Libs -y --no-install-recommends
RUN apt-get update \
    && apt-get install -y \
        curl wget git zip unzip less vim procps lsof tcpdump htop openssl \
        ssh \
        supervisor \
        strace \
        net-tools \
        libz-dev \
        libssl-dev \
        libnghttp2-dev \
        libpcre3-dev \
        libjpeg-dev \
        libpng-dev \
        libfreetype6-dev \
# Install PHP extensions
    && docker-php-ext-install \
       pcntl bcmath gd pdo_mysql mbstring sockets zip sysvmsg sysvsem sysvshm

# Install composer
Run wget https://mirrors.aliyun.com/composer/composer.phar \
    && mv composer.phar /usr/local/bin/composer \
    && chmod u+x /usr/local/bin/composer \
    && composer self-update --clean-backups \

# Install redis extension
    && wget http://pecl.php.net/get/redis-${PHPREDIS_VERSION}.tgz -O /tmp/redis.tar.tgz \
    && pecl install /tmp/redis.tar.tgz \
    && rm -rf /tmp/redis.tar.tgz \
    && docker-php-ext-enable redis \

# Install xlswriter extension
    && curl -fsSL "https://pecl.php.net/get/xlswriter-${XLSWRITER_VERSION}.tgz" -o xlswriter.tgz \
    && mkdir -p /tmp/xlswriter \
    && tar -xf xlswriter.tgz -C /tmp/xlswriter --strip-components=1 \
    && rm xlswriter.tgz \
    && cd /tmp/xlswriter \
    && phpize && ./configure --enable-reader && make && make install \
    && docker-php-ext-enable xlswriter \


# Install swoole extension
    && wget https://github.com/swoole/swoole-src/archive/v${SWOOLE_VERSION}.tar.gz -O swoole.tar.gz \
    && mkdir -p swoole \
    && tar -xf swoole.tar.gz -C swoole --strip-components=1 \
    && rm swoole.tar.gz \
    && ( \
        cd swoole \
        && phpize \
        && ./configure --enable-mysqlnd --enable-sockets --enable-openssl --enable-http2 \
        && make -j$(nproc) \
        && make install \
    ) \
    && rm -r swoole \
    && docker-php-ext-enable swoole \
# Clear dev deps
    && apt-get clean \
    && apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false \

# Timezone
    && cp /usr/share/zoneinfo/${TIMEZONE} /etc/localtime \
    && echo "${TIMEZONE}" > /etc/timezone \
    && echo "[Date]\ndate.timezone=${TIMEZONE}" > /usr/local/etc/php/conf.d/timezone.ini

EXPOSE 9501-9505

# ENTRYPOINT ["php", "/var/www/swoft/bin/swoft", "http:start"]
# CMD ["php", "/var/www/swoft/bin/swoft", "http:start"]
# CMD ["/bin/bash"]






########.bashrc
#phpsw () {
#        app_dir=$(basename `pwd`)
#        tty=
#        tty -s && tty=--tty
#        docker exec \
#        $tty \
#        --interactive \
#        --workdir /usr/share/nginx/html/$app_dir \
#        docker-swoole "$@"
#}


