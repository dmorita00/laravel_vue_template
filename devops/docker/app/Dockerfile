# https://github.com/laradock/php-fpm
# https://github.com/laradock/laradock
# https://unskilled.site/docker%E3%81%AE%E5%85%AC%E5%BC%8Fphp%E3%81%AEdockerfile%E3%82%92%E9%A0%91%E5%BC%B5%E3%81%A3%E3%81%A6%E8%AA%AD%E3%82%93%E3%81%A7%E7%90%86%E8%A7%A3%E3%81%97%E3%82%88%E3%81%86%E3%81%A8%E3%81%97%E3%81%A6/

FROM php:7.4-fpm

ARG PROJECT
ARG DEPLOY
ARG APP_ENV

ENV ACCEPT_EULA=Y

RUN set -eux \
 && apt-get update

RUN set -eux \
 && apt-get install -y --no-install-recommends \
   procps \
   curl \
   git \
   zip \
   unzip \
   libmemcached-dev \
   libz-dev \
   libpq-dev \
   libjpeg-dev \
   libpng-dev \
   libfreetype6-dev \
   libssl-dev \
   libmcrypt-dev \
   dnsutils \
   iputils-ping \
   libzip-dev \
   time \
   zlib1g-dev \
   gnupg \
   libjpeg62-turbo-dev \
   libaio1 \
   wget

RUN set -eux \
 && curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
# && composer config -g repositories.packagist composer https://packagist.jp \
# && composer global require hirak/prestissimo

ENV LD_LIBRARY_PATH /usr/local/instantclient/
ENV TNS_ADMIN /usr/local/instantclient/

# ORACLE oci
ADD devops/docker/app/instantclient-basic-linux.x64-12.1.0.2.0.zip /tmp
ADD devops/docker/app/instantclient-sdk-linux.x64-12.1.0.2.0.zip /tmp

# Install Oracle Instantclient
RUN unzip /tmp/instantclient-basic-linux.x64-12.1.0.2.0.zip -d /usr/local/ \
  && unzip /tmp/instantclient-sdk-linux.x64-12.1.0.2.0.zip -d /usr/local/ \
  && ln -s /usr/local/instantclient_12_1 /usr/local/instantclient \
  && ln -s /usr/local/instantclient/libclntsh.so.12.1 /usr/local/instantclient/libclntsh.so \
  && ln -s /usr/local/instantclient/libclntshcore.so.12.1 /usr/local/instantclient/libclntshcore.so \
  && ln -s /usr/local/instantclient/libocci.so.12.1 /usr/local/instantclient/libocci.so \
  && export LD_LIBRARY_PATH=/usr/local/instantclient \
  && docker-php-ext-configure oci8 --with-oci8=instantclient,/usr/local/instantclient \
  && docker-php-ext-install oci8 \
  && docker-php-ext-configure pdo_oci --with-pdo-oci=instantclient,/usr/local/instantclient,12.1 \
  && docker-php-ext-install pdo_oci \
  && rm -rf /usr/local/*.zip

RUN set -eux \
 && curl https://packages.microsoft.com/keys/microsoft.asc | apt-key add -

RUN set -eux \
 && curl https://packages.microsoft.com/config/debian/10/prod.list > /etc/apt/sources.list.d/mssql-release.list

RUN set -eux \
 && apt-get update

RUN set -eux \
 && apt-get install -y --no-install-recommends \
   unixodbc-dev \
   msodbcsql17 \
   mssql-tools

RUN set -eux \
 # pdo_mysql
 && docker-php-ext-install pdo_mysql \
 # mongodb
# && pecl install mongodb \
# && docker-php-ext-enable mongodb \
 # xdebug
 && if [ "$DEPLOY" = "local" ] ; then \
      pecl install xdebug; \
      docker-php-ext-enable xdebug; \
    fi \
 # GD
 && docker-php-ext-configure gd --with-freetype --with-jpeg \
 && docker-php-ext-install gd \
 # opcache
 && docker-php-ext-configure opcache --enable-opcache \
 && docker-php-ext-install opcache \
 # zip
 && docker-php-ext-install zip
 # SQLServer
# && pecl install sqlsrv \
# && pecl install pdo_sqlsrv \
# && docker-php-ext-enable sqlsrv \
# && docker-php-ext-enable pdo_sqlsrv \
# && docker-php-ext-install bcmath

COPY devops/docker/app /code/devops/docker/app

RUN set -eux \
 && cd /code/devops/docker/app \
 && cp __usr__local__etc__php__conf.d__${DEPLOY}.ini /usr/local/etc/php/conf.d/local.ini

COPY server/composer.json /code/server/composer.json
COPY server/composer.lock /code/server/composer.lock

RUN set -eux \
 && cd /code/server \
 && composer install --no-scripts --no-autoloader
# composer install --no-dev;

COPY server /code/server

RUN set -eux \
 && cd /code/server \
 && composer dumpautoload -o \
 # localの場合はcacheを作らない。
 # cacheすると.env.testingが読まれない不具合が発生した。
 && if [ "$DEPLOY" != "local" ] ; then \
      php artisan config:cache; \
      php artisan route:cache; \
    fi \
 && chown -R www-data:www-data storage \
# && chown -R www-data:www-data resources/data \
 # tinkerで /var/www/.configに書き込むので
 && chown -R www-data:www-data /var/www

USER www-data

WORKDIR /code/server
