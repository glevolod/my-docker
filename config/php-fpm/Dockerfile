FROM php:7.4-fpm

ARG LOCAL_USER_ID
RUN useradd proxy_user -m -u $LOCAL_USER_ID

COPY wait-for-it.sh /usr/bin/wait-for-it
RUN chmod +x /usr/bin/wait-for-it

# netcat needed for executing nc command by wait-for-it
RUN apt-get -q update && apt-get -qy install netcat libzip-dev \
    && docker-php-ext-install pdo_mysql \
    && docker-php-ext-install zip

ARG XDEBUG_REMOTE_HOST
ARG WITH_XDEBUG
ARG APP_ENV
RUN if [ ${APP_ENV} = "dev" ] ; \
    then \
        cp "$PHP_INI_DIR/php.ini-development" "$PHP_INI_DIR/php.ini"; \
    else \
        cp "$PHP_INI_DIR/php.ini-production" "$PHP_INI_DIR/php.ini" ; \
    fi ;

RUN if [ ${WITH_XDEBUG} = "true" ] ; \
    then \
        pecl install xdebug-2.9.3; \
        docker-php-ext-enable xdebug; \
        echo "xdebug.remote_enable=1" >> $PHP_INI_DIR/conf.d/docker-php-ext-xdebug.ini; \
        echo "xdebug.remote_autostart=1" >> $PHP_INI_DIR/conf.d/docker-php-ext-xdebug.ini; \
        echo "xdebug.remote_host=${XDEBUG_REMOTE_HOST}" >> $PHP_INI_DIR/conf.d/docker-php-ext-xdebug.ini; \
    fi ;
COPY conf.d/custom.ini $PHP_INI_DIR/conf.d/99-custom.ini

COPY --from=composer:1.10.1 /usr/bin/composer /usr/bin/composer

WORKDIR /var/www

CMD wait-for-it db:3306 -- php-fpm

EXPOSE 9000
