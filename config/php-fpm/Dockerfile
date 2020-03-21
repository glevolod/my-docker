FROM php:7.4-fpm

COPY wait-for-it.sh /usr/bin/wait-for-it

RUN chmod +x /usr/bin/wait-for-it

# netcat needed for executing nc command by wait-for-it
RUN apt-get -q update && apt-get -qy install netcat \
    && docker-php-ext-install pdo_mysql \
    && pecl install xdebug-2.9.3 \
    && docker-php-ext-enable xdebug

RUN cp "$PHP_INI_DIR/php.ini-development" "$PHP_INI_DIR/php.ini"

ARG XDEBUG_REMOTE_HOST
COPY conf.d/custom.ini $PHP_INI_DIR/conf.d/
RUN sed -i "s/__remote_host__/${XDEBUG_REMOTE_HOST}/g" $PHP_INI_DIR/conf.d/custom.ini

COPY --from=composer:1.10.1 /usr/bin/composer /usr/bin/composer

WORKDIR /var/www

CMD wait-for-it database:3306 -- php-fpm

EXPOSE 9000
