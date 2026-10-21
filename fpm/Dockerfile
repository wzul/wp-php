# Patched PHP-FPM for the flyops fleet, built from the official php.net tarball.
# Used while Docker Hub's own image for a security release has not landed yet.
ARG BASE=php:8.5-fpm-alpine
FROM ${BASE}

ARG PHP_VERSION=8.5.12

RUN apk add --no-cache --virtual .build-deps $PHPIZE_DEPS linux-headers \
      libjpeg-turbo-dev libpng-dev freetype-dev libwebp-dev \
      libzip-dev icu-dev oniguruma-dev libxml2-dev libxslt-dev \
      postgresql-dev sqlite-dev libsodium-dev openldap-dev \
      curl-dev openssl-dev zlib-dev gettext-dev gmp-dev

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
      --with-iconv --with-gettext --with-gmp --with-ldap; \
    make -j"$(nproc)"; \
    make install; \
    strip /usr/local/bin/php /usr/local/sbin/php-fpm; \
    php -v

COPY --from=mlocati/php-extension-installer:latest /usr/bin/install-php-extensions /usr/local/bin/

# Runtime libraries the compiled PHP links against. They must be installed as
# explicit dependencies: the -dev packages above live in .build-deps and taking
# that virtual package away also takes libzip/libpng/icu with it, which leaves
# /usr/local/bin/php unable to relocate its symbols.
RUN apk add --no-cache ghostscript imagemagick msmtp \
      libzip libpng libjpeg-turbo freetype libwebp icu-libs oniguruma libxml2 \
      libxslt postgresql-libs sqlite-libs libsodium openldap libcurl zlib \
      gettext-libs gmp libstdc++

RUN install-php-extensions redis apcu igbinary msgpack imagick \
 && rm -rf /tmp/php.tar.xz /usr/src/php-new \
 && gs --version \
 && php -r 'echo "BUILT " . PHP_VERSION . "\n";' \
 && php -m | tr '\n' ' '
