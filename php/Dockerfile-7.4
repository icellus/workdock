FROM php:7.4-fpm

ENV DEBIAN_FRONTEND noninteractive

USER root

#    sed -i 's/deb.debian.org/mirrors.tuna.tsinghua.edu.cn/' /etc/apt/sources.list && \
#    sed -i 's/security.debian.org/mirrors.tuna.tsinghua.edu.cn/' /etc/apt/sources.list && \
#    sed -i 's/security-cdn.debian.org/mirrors.tuna.tsinghua.edu.cn/' /etc/apt/sources.list
RUN sed -i 's/deb.debian.org/mirrors.aliyun.com/' /etc/apt/sources.list && \
    sed -i 's/security.debian.org/mirrors.aliyun.com/' /etc/apt/sources.list && \
    sed -i 's/security-cdn.debian.org/mirrors.aliyun.com/' /etc/apt/sources.list;

RUN ln -snf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && echo 'Asia/Shanghai'> /etc/timezone

RUN set -eux; \
    apt-get update; \
    apt-get upgrade -y; \
    pecl channel-update pecl.php.net && \
    groupadd -g 1000 www && \
    useradd -u 1000 -g www -m www && \
    usermod -p "*" www -s /bin/bash && \
    apt-get install -yqq --no-install-recommends \
            apt-utils \
            curl \
            libmemcached-dev \
            libz-dev \
            libpq-dev \
            libjpeg-dev \
            libpng-dev \
            libwebp-dev \
            libfreetype6-dev \
            libssl-dev \
            libmcrypt-dev \
            libzip-dev zip unzip \
            libonig-dev \
            inetutils-ping \
            vim \
            git \
            wget \
            locales-all \
            procps;

# Installing tools and PHP extentions using "apt", "docker-php", "pecl",
USER root

RUN set -eux; \
    pecl install -o -f redis xlswriter; \
    docker-php-ext-configure zip; \
    docker-php-ext-install zip; \
    docker-php-ext-install mysqli; \
    # Install the PHP pdo_mysql extention
    docker-php-ext-install pdo_mysql; \
    # Install the PHP gd library
    docker-php-ext-configure gd \
            --prefix=/usr \
            --with-jpeg \
            --with-webp \
            --with-freetype; \
    docker-php-ext-install bcmath; \
    docker-php-ext-install gd;  \
    docker-php-ext-install opcache; \
    docker-php-ext-install pcntl; \
    docker-php-ext-enable redis; \
    docker-php-ext-enable xlswriter

# intl  CI Framework require
ARG INSTALL_INTL=false

RUN if [ ${INSTALL_INTL} = true ]; then \
    # Install intl and requirements
    apt-get install -y zlib1g-dev libicu-dev g++ && \
    docker-php-ext-configure intl && \
    docker-php-ext-install intl \
;fi

# todo xDebug

# todo event 扩展 workman required pcntl posix event|libevent

# swoole
#RUN set -eux; \
#      # Install Php Swoole Extension
#      if   [ $(php -r "echo PHP_VERSION_ID - PHP_RELEASE_VERSION;") = "50600" ]; then \
#        pecl install swoole-2.0.10; \
#      elif [ $(php -r "echo PHP_VERSION_ID - PHP_RELEASE_VERSION;") = "70000" ]; then \
#        pecl install swoole-4.3.5; \
#      elif [ $(php -r "echo PHP_VERSION_ID - PHP_RELEASE_VERSION;") = "70100" ]; then \
#        pecl install swoole-4.5.11; \
#      elif [ $(php -r "echo PHP_MAJOR_VERSION;") = "7" ]; then \
#        pecl install swoole-4.8.12; \
#      else \
#        pecl install swoole; \
#      fi; \
#      docker-php-ext-enable swoole; \
#      php -m | grep -q 'swoole'; \

RUN set -xe; php -v | head -n 1

# Composer:
#   http://getcomposer.org/installer    国外源
#   https://install.phpcomposer.com/installer   国内源
ARG COMPOSER_INSTALLER_REPO=https://install.phpcomposer.com/installer

RUN curl -s $COMPOSER_INSTALLER_REPO | php && \
    mv composer.phar /usr/local/bin/composer && \
    composer config -g repo.packagist composer https://mirrors.aliyun.com/composer

USER www

RUN composer config -g repo.packagist composer https://mirrors.aliyun.com/composer

# final:
USER root

COPY ./conf/opcache.ini /usr/local/etc/php/conf.d/opcache.ini
COPY ./conf/php-user.ini /usr/local/etc/php/conf.d/
COPY ./conf/xphp-fpm.conf /usr/local/etc/php-fpm.d/
COPY ./conf/vimrc.local /etc/vim/
COPY ./conf/aliases.sh /root/aliases.sh

RUN sed -i 's/\r//' /root/aliases.sh && \
    echo "" >> ~/.bashrc && \
    echo "source ~/aliases.sh" >> ~/.bashrc && \
	echo "" >> ~/.bashrc && \
	cp ~/aliases.sh /home/www/aliases.sh && \
    chown www:www /home/www/aliases.sh


# Clean up
RUN apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
    rm -f /var/log/lastlog /var/log/faillog

WORKDIR /var/www

CMD ["php-fpm"]

EXPOSE 9000