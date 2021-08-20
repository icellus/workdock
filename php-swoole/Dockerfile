FROM php:7.4-cli-alpine

# Change application source from dl-cdn.alpinelinux.org to aliyun source
ARG CHANGE_SOURCE=false
RUN if [ ${CHANGE_SOURCE} = true ]; then \
    sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/' /etc/apk/repositories \
;fi

RUN set -ex \
    && apk update \
    && apk add --no-cache \
    # Install base packages ('ca-certificates' will install 'nghttp2-libs')
    ca-certificates \
    ffmpeg \
    curl \
    wget \
    vim \
    tar \
    xz \
    libressl \
    tzdata \
    pcre \
    freetype \
    freetype-dev \
    libpng \
    libpng-dev \
    libjpeg-turbo \
    libjpeg-turbo-dev \
    libzip \
    libzip-dev \
    && docker-php-ext-configure gd \
                --prefix=/usr \
                --with-jpeg \
                --with-freetype \
    && docker-php-ext-install bcmath \
        gd \
        pcntl \
        pdo_mysql \
        sockets \
        sysvmsg \
        sysvsem \
        sysvshm \
        opcache \
        zip \
    && php -v \
    && php -m \
    && apk del --purge *-dev \
    && rm -rf /var/cache/apk/* /tmp/* /usr/share/man

###########################################################################
# Composer:
###########################################################################
# Install composer and add its bin to the PATH.
#   http://getcomposer.org/installer    国外源
#   https://install.phpcomposer.com/installer   国内源
ARG COMPOSER_INSTALLER_REPO
ENV COMPOSER_INSTALLER_REPO ${COMPOSER_INSTALLER_REPO}

# install composer
# wget -nv -O /usr/local/bin/composer https://github.com/composer/composer/releases/download/${COMPOSER_VERSION}/composer.phar \
# chmod u+x /usr/local/bin/composer
RUN curl -s $COMPOSER_INSTALLER_REPO | php \
    && mv composer.phar /usr/local/bin/composer \
    && chmod u+x /usr/local/bin/composer \
    && composer config -g repo.packagist composer https://mirrors.aliyun.com/composer \
    && composer

###########################################################################
# redis|swoole|wxwork_finance_sdk
###########################################################################
# ---------- env settings ----------
ENV PHPIZE_DEPS="autoconf dpkg-dev dpkg file g++ gcc libc-dev make php7-dev php7-pear pkgconf re2c pcre-dev pcre2-dev zlib-dev libtool automake"

COPY ./swoole-4.6.3.tar.gz /tmp/swoole.tar.gz
COPY ./wework-chatdata-sdk-0.1.0.tar.gz /tmp/wxwork_finance_sdk.tar.gz

RUN set -ex \
    && apk update \
    # for swoole extension libaio linux-headers
    && apk add --no-cache libstdc++ openssl git bash \
    && apk add --no-cache --virtual .build-deps $PHPIZE_DEPS libaio-dev openssl-dev curl-dev \
    # php extension:redis
    && pecl channel-update pecl.php.net \
    && pecl install -o -f redis \
    && docker-php-ext-enable redis \
    # php extension:swoole
    # download
    && cd /tmp \
    && curl -SL "https://github.com/swoole/swoole-src/archive/${SW_VERSION}.tar.gz" -o swoole.tar.gz \
    && ls -alh \
    && cd /tmp \
    && mkdir -p swoole \
    && tar -xf swoole.tar.gz -C swoole --strip-components=1 \
    && ( \
        cd swoole \
        && phpize \
        && ./configure --enable-openssl --enable-http2 --enable-swoole-curl --enable-swoole-json \
        && make -s -j$(nproc) && make install \
    ) \
    && echo "memory_limit=1G" > /usr/local/etc/php/conf.d/00_default.ini \
    && echo "opcache.enable_cli = 'On'" >> /usr/local/etc/php/conf.d/00_opcache.ini \
    && echo "extension=swoole.so" > /usr/local/etc/php/conf.d/50_swoole.ini \
    && echo "swoole.use_shortname = 'Off'" >> /usr/local/etc/php/conf.d/50_swoole.ini \
    # php extension:wxwork_finance_sdk
#    && ln -s /lib/libc.musl-x86_64.so.1 /lib/ld-linux-x86-64.so.2 \
#    && cd /tmp \
#   # && curl -SL "https://github.com/oh-stone/wework-chatdata-sdk/archive/v0.1.0.tar.gz" -o wxwork_finance_sdk.tar.gz \
#    && mkdir -p wxwork_finance_sdk \
#    && tar -xf wxwork_finance_sdk.tar.gz -C wxwork_finance_sdk --strip-components=1 \
#    && ( \
#      cd wxwork_finance_sdk/php7-wxwork-finance-sdk \
#      && phpize \
#      && ./configure --with-wxwork-finance-sdk=/tmp/wxwork_finance_sdk/C_sdk \
#      && make && make install \
#    ) \
#    && cp /tmp/wxwork_finance_sdk/C_sdk/libWeWorkFinanceSdk_C.so /usr/local/lib/libWeWorkFinanceSdk_C.so \
#    && echo "extension=wxwork_finance_sdk.so" > /usr/local/etc/php/conf.d/50_wxwork_finance_sdk.ini \
    # php info
    && php -v \
    && php -m \
    && php --ri redis \
    && php --ri swoole \
#    && php --ri wxwork_finance_sdk \
    && apk del .build-deps \
    && rm -rf /var/cache/apk/* /tmp/* /usr/share/man
