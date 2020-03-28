# docker-compose config stub for nginx+mysql+php-fpm
This config provide ability to run environment for php applications with Nginx, MySQL 5.7, php-fpm 7.4 in three appropriate containers

## How to use
1. clone this repo
1. create **app** directory in root path of this repo and put your application (Symfony) there
1. copy **.env.example** into **.env** and change values
1. edit **/etc/hosts** for resolving host from **HOST** value of **.env** file into **127.0.0.1**
1. execute `docker-compose up` in **config** directory in console

## Explanation
Configuration tested with Docker 19.03.8, Docker Compose 1.25.4
#### Nginx
Nginx config is suitable for Symfony 5 but can be adopted for any project (Laravel, Yii, Zend etc.)\
Nginx logs are stored in logs/nginx folder 
#### MySQL
MySQL data and tables are stored in /config/database/data folder \
Users and database created when container is built (**DATABASE_**\* variables in **.env** file)

#### Php-fpm
php-fpm container uses **wait-for-it.sh** script to detect if database is ready for connections \
php.ini variables can be changed in /config/php-fpm/conf.d/custom.ini \
If **APP_ENV=dev** in **.env** file, then php.ini-development config is used, php.ini-production instead \
If **WITH_XDEBUG=true** in **.env** file, then container is built with xdebug-2.9.3 . 
For configuring xdebug **XDEBUG_REMOTE_HOST** will be set as a network bridge used for docker. \
For debugging cli in PHPStorm **PHP_IDE_CONFIG=serverName=php-fpm** will be changed in **.env** file.
Where **'php-fpm'** is a name of server from "Settings > Languages & Frameworks > PHP > Servers" in PHPStorm \

##### Improved permissions for php container
During php container creation user **proxy_user** is added in it with id as you can specify in **LOCAL_USER_ID** 
variable of **.env** file.
Usually id will be set in 1000 (as a first user id for login in linux). So this user will be used when you enter into 
php container (like 'docker exec -it my-docker_php_1 bash') for executing console commands (like 'composer install').
And so new files created such way in the php container will have the same user:group permissions as you need in your 
host machine.

#### Database management
'adminer' image is used for database management. Interface is available on port 8080 on localhost