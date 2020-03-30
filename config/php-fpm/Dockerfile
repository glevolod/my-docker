FROM php:7.4-fpm

ARG LOCAL_USER_ID
RUN useradd proxy_user -m -u $LOCAL_USER_ID

COPY wait-for-it.sh /usr/bin/wait-for-it
RUN chmod +x /usr/bin/wait-for-it

# netcat needed for executing nc command by wait-for-it
RUN apt-get -q update && apt-get -qy install netcat libzip-dev libicu-dev libonig-dev libcurl4-openssl-dev \
    && docker-php-ext-install pdo_mysql pdo zip opcache intl mbstring curl exif json calendar fileinfo \
#    bcmath bz2 ctype dba dom enchant ffi filter ftp gd gettext gmp hash iconv imap ldap mysqli oci8 odbc pcntl pdo_dblib \
#    pdo_firebird pdo_oci pdo_odbc pdo_pgsql pdo_sqlite pgsql phar posix pspell readline reflection session shmop \
#    simplexml snmp soap sockets sodium spl standard sysvmsg sysvsem sysvshm tidy tokenizer xml xmlreader xmlrpc xmlwriter xsl zend_test \
;

RUN apt-get -qy install git \
    && git config --system alias.hist "log --pretty=format:'%h %ad | %s%d [%an]' --graph --date=short" \
    && git config --system core.excludesfile ~/.gitignore

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
RUN curl https://get.symfony.com/cli/installer -o - | bash \
    && mv /root/.symfony/bin/symfony /usr/local/bin/symfony

WORKDIR /var/www

CMD wait-for-it db:3306 -- php-fpm

EXPOSE 9000
