ARG PHP_VERSION=8.3

FROM composer:2 AS composer

FROM php:${PHP_VERSION}-fpm-bookworm AS base

ARG UID=1000
ARG GID=1000

# Adjust apt packages and PHP extensions to what composer.json actually requires.
RUN apt-get update && apt-get install -y --no-install-recommends \
        acl \
        curl \
        git \
        libicu-dev \
        libzip-dev \
        unzip \
    && docker-php-ext-install -j"$(nproc)" \
        intl \
        opcache \
        pdo_mysql \
        zip \
    && rm -rf /var/lib/apt/lists/*

# Uncomment only the extensions the project actually needs:
# RUN docker-php-ext-configure gd --with-freetype --with-jpeg \
#     && docker-php-ext-install -j"$(nproc)" gd
# RUN pecl install redis && docker-php-ext-enable redis
# RUN pecl install amqp && docker-php-ext-enable amqp

COPY --from=composer /usr/bin/composer /usr/bin/composer

RUN groupmod -o -g "${GID}" www-data \
    && usermod -o -u "${UID}" -g "${GID}" www-data \
    && mkdir -p /var/www/html/vendor \
    && chown -R www-data:www-data /var/www/html \
    && git config --system --add safe.directory /var/www/html

COPY docker/php/conf.d/symfony.ini /usr/local/etc/php/conf.d/zz-symfony.ini
COPY docker/php/docker-entrypoint.sh /usr/local/bin/docker-entrypoint.sh
RUN chmod +x /usr/local/bin/docker-entrypoint.sh

WORKDIR /var/www/html
USER www-data

ENTRYPOINT ["docker-entrypoint.sh"]
CMD ["php-fpm"]

FROM base AS dev

USER root
RUN pecl install xdebug \
    && docker-php-ext-enable xdebug
COPY docker/php/conf.d/xdebug.ini /usr/local/etc/php/conf.d/zz-xdebug.ini
USER www-data
