FROM php:8.1-fpm

COPY wait-for-it.sh /usr/bin/wait-for-it
RUN chmod +x /usr/bin/wait-for-it

# netcat needed for executing nc command by wait-for-it
RUN apt-get update && apt-get install -y mc netcat libzip-dev libicu-dev libonig-dev libcurl4-openssl-dev zip unzip curl \
    gnupg gnupg2 gnupg1 nano git \
    && git config --system alias.hist "log --pretty=format:'%h %ad | %s%d [%an]' --graph --date=short" \
    && git config --system core.excludesfile /home/proxy_user/.global_gitignore

COPY --from=mlocati/php-extension-installer /usr/bin/install-php-extensions /usr/local/bin/
RUN install-php-extensions intl opcache pdo_mysql \
#    bcmath bz2 ctype dba dom enchant ffi filter ftp gd gettext gmp hash iconv imap ldap mysqli oci8 odbc pcntl pdo_dblib \
#    pdo_firebird pdo_oci pdo_odbc pdo_pgsql pdo_sqlite pgsql phar posix pspell readline reflection session shmop \
#    simplexml snmp soap sockets sodium spl standard sysvmsg sysvsem sysvshm tidy tokenizer xml xmlreader xmlrpc xmlwriter xsl zend_test \
;

RUN curl -fsSL https://deb.nodesource.com/setup_18.x | bash -
RUN apt-get install -y nodejs

RUN echo 'deb [trusted=yes] https://repo.symfony.com/apt/ /' | tee /etc/apt/sources.list.d/symfony-cli.list
RUN apt-get update && apt-get install -y symfony-cli

RUN curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add -
RUN echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list
RUN apt-get update && apt-get install -y yarn

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
        install-php-extensions xdebug-^3.1; \
        echo "xdebug.mode=develop,debug" >> $PHP_INI_DIR/conf.d/docker-php-ext-xdebug.ini; \
        echo "xdebug.start_with_request=1" >> $PHP_INI_DIR/conf.d/docker-php-ext-xdebug.ini; \
        echo "xdebug.client_host=${XDEBUG_REMOTE_HOST}" >> $PHP_INI_DIR/conf.d/docker-php-ext-xdebug.ini; \
    fi ;
COPY conf.d/custom.ini $PHP_INI_DIR/conf.d/99-custom.ini

COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

WORKDIR /var/www

ENV LANG C.UTF-8
ENV LC_ALL C.UTF-8

ENV HISTFILE /home/proxy_user/stored/.bash_history

ARG LOCAL_USER_ID
RUN useradd proxy_user -m -u $LOCAL_USER_ID
RUN mkdir /home/proxy_user/.ssh \
    && ssh-keyscan github.com >> /home/proxy_user/.ssh/known_hosts \
    && chown -R proxy_user:proxy_user /home/proxy_user/.ssh

CMD wait-for-it db:3306 -- php-fpm

USER proxy_user
EXPOSE 9000
