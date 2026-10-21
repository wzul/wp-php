# Patched PHP-FPM for the flyops fleet.
# Upstream images lag the security release (the PHP2Shell fix shipped in source
# only), so the core is rebuilt from the official tarball at the same minor
# version: module ABI is unchanged, extensions compile normally.
ARG BASE=php:8.5-fpm-alpine
FROM ${BASE}

ARG PHP_VERSION=8.5.12

RUN apk add --no-cache --virtual .build-deps $PHPIZE_DEPS \
 && apk add --no-cache \
      libjpeg-turbo-dev libpng-dev freetype-dev libwebp-dev \
      libzip-dev icu-dev oniguruma-dev libxml2-dev libxslt-dev libssh2-dev \
      postgresql-dev sqlite-dev libsodium-dev imagemagick-dev openldap-dev curl-dev openssl-dev zlib-dev gettext-dev gmp-dev

RUN set -eux; \
    curl -fsSL "https://www.php.net/distributions/php-${PHP_VERSION}.tar.xz" -o /tmp/php.tar.xz; \
    mkdir -p /usr/src/php-new; \
    tar -xJf /tmp/php.tar.xz -C /usr/src/php-new --strip-components=1; \
    cd /usr/src/php-new; \
    ./configure \
      --build="$(apk --print-arch)" \
      --with-config-file-path=/usr/local/etc/php \
      --with-config-file-scan-dir=/usr/local/etc/php/conf.d \
      --enable-fpm --with-fpm-user=www-data --with-fpm-group=www-data \
      --enable-mbstring --enable-intl --enable-soap --enable-sockets --enable-bcmath \
      --enable-calendar --enable-exif --enable-pcntl \
      --enable-shmop --enable-sysvsem --enable-opcache --with-sodium \
      --with-curl --with-openssl --with-zlib --with-zip --with-libxml --with-xsl \
      --with-gd --with-jpeg --with-freetype --with-webp \
      --with-pdo-mysql=mysqlnd --with-mysqli=mysqlnd --enable-mysqlnd \
      --with-pdo-pgsql --with-pgsql --with-pdo-sqlite --with-sqlite3 \
      --with-iconv --with-gettext --with-gmp --with-ldap \
    && make -j"$(nproc)" \
    && make install \
    && strip /usr/local/bin/php /usr/local/sbin/php-fpm \
    && php -v

RUN docker-php-ext-install -j"$(nproc)" pdo_mysql mysqli gd intl zip exif bcmath opcache \
 && pecl install redis apcu igbinary msgpack imagick \
 && docker-php-ext-enable redis apcu igbinary msgpack imagick \
 && apk del .build-deps \
 && rm -rf /tmp/php.tar.xz /usr/src/php-new \
 && php -m | tr '\n' ' ' && php -r 'echo "\nBUILT " . PHP_VERSION . "\n";'
