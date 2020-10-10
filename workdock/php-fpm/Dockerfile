FROM php:7.4-fpm

# Set Environment Variables
ENV DEBIAN_FRONTEND noninteractive

ARG CHANGE_SOURCE=false
RUN if [ ${CHANGE_SOURCE} = true ]; then \
    # Change application source from deb.debian.org to aliyun source
    sed -i 's/deb.debian.org/mirrors.aliyun.com/' /etc/apt/sources.list && \
    sed -i 's/security.debian.org/mirrors.aliyun.com/' /etc/apt/sources.list && \
    sed -i 's/security-cdn.debian.org/mirrors.aliyun.com/' /etc/apt/sources.list \
;fi

#
#--------------------------------------------------------------------------
# Software's Installation
#--------------------------------------------------------------------------
#
# Installing tools and PHP extentions using "apt", "docker-php", "pecl",
#
RUN set -eux; \
    apt-get update; -yqq \
    apt-get upgrade -y; \
    pecl channel-update pecl.php.net && \
    apt-get install -yqq --no-install-recommends \
            apt-utils \
            curl \
            libmemcached-dev \
            libz-dev \
            libpq-dev \
            libjpeg-dev \
            libpng-dev \
            libfreetype6-dev \
            libssl-dev \
            libmcrypt-dev \
            libzip-dev zip unzip \
            libonig-dev \
            vim \
            git \
            wget \
            procps;

RUN set -eux; \
    # Install the PHP pdo_mysql extention
    if [ $(php -r "echo PHP_MAJOR_VERSION;") = "5" ]; then \
        # Install the PHP mcrypt extention
        docker-php-ext-install mcrypt; \
        # Install the PHP msyqli extention
        docker-php-ext-install mysqli; \
        # Install the PHP gd library
        docker-php-ext-configure gd \
                --enable-gd-native-ttf \
                --with-jpeg-dir=/usr/lib \
                --with-freetype-dir=/usr/include/freetype2; \
        docker-php-ext-install gd; \
        docker-php-ext-configure zip --with-libzip; \
    else \
        # Install the PHP gd library
        docker-php-ext-configure gd \
                --prefix=/usr \
                --with-jpeg \
                --with-freetype; \
        docker-php-ext-install gd ;\
        docker-php-ext-configure zip; \
    fi; \
    docker-php-ext-install zip; \
    docker-php-ext-install pdo_mysql;

# Install composer and add its bin to the PATH.
#   http://getcomposer.org/installer    国外源
#   https://install.phpcomposer.com/installer   国内源
RUN if [ ${CHANGE_SOURCE} = true ]; then \
        curl -s https://install.phpcomposer.com/installer | php && \
    else \
        curl -s http://getcomposer.org/installer | php && \
    ;fi
    echo "export PATH=${PATH}:/var/www/vendor/bin" >> ~/.bashrc && \
    mv composer.phar /usr/local/bin/composer

# Source the bash
RUN . ~/.bashrc

RUN composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/

###########################################################################
# xDebug:
###########################################################################
ARG INSTALL_XDEBUG=false

RUN if [ ${INSTALL_XDEBUG} = true ]; then \
  # Install the xdebug extension
  if [ $(php -r "echo PHP_MAJOR_VERSION;") = "5" ]; then \
    pecl install xdebug-2.5.5; \
  else \
    if [ $(php -r "echo PHP_MINOR_VERSION;") = "0" ]; then \
      pecl install xdebug-2.9.0; \
    else \
      pecl install xdebug; \
    fi \
  fi && \
  docker-php-ext-enable xdebug \
;fi

# Copy xdebug configuration for remote debugging
COPY ./xdebug.ini /usr/local/etc/php/conf.d/xdebug.ini

RUN sed -i "s/xdebug.remote_autostart=0/xdebug.remote_autostart=1/" /usr/local/etc/php/conf.d/xdebug.ini && \
    sed -i "s/xdebug.remote_enable=0/xdebug.remote_enable=1/" /usr/local/etc/php/conf.d/xdebug.ini && \
    sed -i "s/xdebug.cli_color=0/xdebug.cli_color=1/" /usr/local/etc/php/conf.d/xdebug.ini

###########################################################################
# PHP REDIS EXTENSION
###########################################################################

ARG INSTALL_PHPREDIS=false

RUN if [ ${INSTALL_PHPREDIS} = true ]; then \
    # Install Php Redis Extension
    if [ $(php -r "echo PHP_MAJOR_VERSION;") = "5" ]; then \
      pecl install -o -f redis-4.3.0; \
    else \
      pecl install -o -f redis; \
    fi \
    && rm -rf /tmp/pear \
    && docker-php-ext-enable redis \
;fi

###########################################################################
# Xhprof:
###########################################################################
ARG INSTALL_XHPROF=false

RUN if [ ${INSTALL_XHPROF} = true ]; then \
    # Install the php xhprof extension
    if [ $(php -r "echo PHP_MAJOR_VERSION;") = 7 ]; then \
      curl -L -o /tmp/xhprof.tar.gz "https://github.com/tideways/php-xhprof-extension/archive/v5.0.1.tar.gz"; \
    else \
      curl -L -o /tmp/xhprof.tar.gz "https://codeload.github.com/phacility/xhprof/tar.gz/master"; \
    fi \
    && mkdir -p xhprof \
    && tar -C xhprof -zxvf /tmp/xhprof.tar.gz --strip 1 \
    && ( \
        cd xhprof \
        && phpize \
        && ./configure \
        && make \
        && make install \
    ) \
    && rm -r xhprof \
    && rm /tmp/xhprof.tar.gz \
;fi

COPY ./xhprof.ini /usr/local/etc/php/conf.d

RUN if [ ${INSTALL_XHPROF} = false ]; then \
    rm /usr/local/etc/php/conf.d/xhprof.ini \
;fi

###########################################################################
# Opcache:
###########################################################################
ARG INSTALL_OPCACHE=false

RUN if [ ${INSTALL_OPCACHE} = true ]; then \
    docker-php-ext-install opcache \
;fi

# Copy opcache configration
COPY ./opcache.ini /usr/local/etc/php/conf.d/opcache.ini

###########################################################################
# ping:
###########################################################################
USER root

ARG INSTALL_PING=false

RUN if [ ${INSTALL_PING} = true ]; then \
    apt-get -y install inetutils-ping \
;fi

###########################################################################
# workman required
###########################################################################
USER root

ARG INSTALL_WORKMAN_REQUIRED=false

RUN if [ ${INSTALL_WORKMAN_REQUIRED} = true ]; then \
    # pcntl
    docker-php-ext-install pcntl \
;fi
###########################################################################
# final:
###########################################################################
ARG TZ=PRC
ENV TZ ${TZ}

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

RUN set -xe; php -v | head -n 1

COPY ./php-user.ini /usr/local/etc/php/conf.d/
COPY ./php-fpm.conf /usr/local/etc/php-fpm.d/

COPY ./conf/vimrc.local /etc/vim/

USER root

# Clean up
RUN apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
    rm /var/log/lastlog /var/log/faillog

# Configure non-root user.
ARG PUID=1000
ENV PUID ${PUID}
ARG PGID=1000
ENV PGID ${PGID}


RUN groupadd -g ${PGID} www && \
    useradd -u ${PUID} -g www -m www
#RUN groupmod -o -g ${PGID} www && \
#    usermod -o -u ${PUID} -g www

# Configure locale.
ARG LOCALE=POSIX
ENV LC_ALL ${LOCALE}

###########################################################################
# User Aliases
###########################################################################
USER root

COPY ./conf/aliases.sh /root/aliases.sh
COPY ./conf/aliases.sh /home/www/aliases.sh

RUN sed -i 's/\r//' /root/aliases.sh && \
    sed -i 's/\r//' /home/www/aliases.sh && \
    chown www:www /home/www/aliases.sh && \
    echo "" >> ~/.bashrc && \
    echo "# Load Custom Aliases" >> ~/.bashrc && \
    echo "source ~/aliases.sh" >> ~/.bashrc && \
	echo "" >> ~/.bashrc


###########################################################################
# composer install function dump|dd
###########################################################################
USER www

RUN echo "" >> ~/.bashrc && \
    echo "# Load Custom Aliases" >> ~/.bashrc && \
    echo "source ~/aliases.sh" >> ~/.bashrc && \
	  echo "" >> ~/.bashrc

RUN composer global require symfony/var-dumper

COPY ./function_dd.php /home/www/.composer

RUN echo ", \"autoload\": {\"files\": [\"/home/www/.composer/function_dd.php\" ]}" >> /home/www/.composer/composer.json

USER root

RUN echo "\n auto_prepend_file=\"/home/www/.composer/vendor/autoload.php\"" >> /usr/local/etc/php/conf.d/php-user.ini

WORKDIR /var/www

CMD ["php-fpm"]

EXPOSE 9000