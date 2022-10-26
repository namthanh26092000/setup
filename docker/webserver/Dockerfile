FROM ubuntu:18.04

ENV PHP_VERSION 7.2
ENV PHP_74 7.4
ENV DEBIAN_FRONTEND=nonintercative

RUN apt-get update
RUN apt-get upgrade -y
RUN apt-get install -y software-properties-common language-pack-en-base
RUN LC_ALL=en_US.UTF-8 add-apt-repository ppa:ondrej/php
RUN apt-get update && apt-get install -y zip nano curl wget

# Install GIT
RUN apt-get install software-properties-common -y
RUN add-apt-repository ppa:git-core/ppa && apt-get update && apt-get install git -y

# Install nginx
COPY config/etc/apt/sources.list.d/nginx-ubuntu-stable-xenial.list /etc/apt/sources.list.d/nginx-ubuntu-stable-xenial.list
RUN apt-key adv --recv-keys --keyserver hkp://keyserver.ubuntu.com 00A6F0A3C300EE8C
RUN apt-get update && apt-get install -y nginx

# Install nodejs
RUN apt-get update && apt-get install -y curl
RUN curl -sL https://deb.nodesource.com/setup_11.x | bash -
RUN apt-get update && apt-get install -y nodejs
RUN npm install -g bower
RUN npm install -g yarn
RUN npm install -g gulp

#Install supervisor
RUN apt-get install supervisor -y

# Copy Config
COPY config/supervisor.conf /etc/supervisor/conf.d/supervisor.conf
COPY config/nginx.conf /etc/nginx/nginx.conf

# Install nginx_ensite
RUN apt-get install -y make
RUN git clone https://github.com/perusio/nginx_ensite.git /tmp/nginx_ensite &&  cd /tmp/nginx_ensite/ && make install && rm -r /tmp/nginx_ensite

RUN wget https://getcomposer.org/download/$(curl -s https://api.github.com/repos/composer/composer/releases/latest | grep 'tag_name' | cut -d '"' -f 4)/composer.phar -O /usr/local/bin/composer && chmod +x /usr/local/bin/composer

# Install php7.2 & module
RUN apt-get install -y php$PHP_VERSION php$PHP_VERSION-cli php$PHP_VERSION-fpm
RUN apt-get install -y php$PHP_VERSION-gd php$PHP_VERSION-json php$PHP_VERSION-mysql
RUN apt-get install -y php$PHP_VERSION-readline php$PHP_VERSION-mbstring
RUN apt-get install -y php$PHP_VERSION-dom php$PHP_VERSION-curl
RUN apt-get install -y php$PHP_VERSION-zip php$PHP_VERSION-redis php$PHP_VERSION-memcached
RUN apt-get install -y php$PHP_VERSION-bcmath php$PHP_VERSION-bz2 php$PHP_VERSION-gmp php$PHP_VERSION-mongo php$PHP_VERSION-intl php$PHP_VERSION-soap php${PHP_VERSION}-amqp

# Install php7.4 & module
#RUN apt-get install -y php$PHP_74 php$PHP_74-cli 
RUN apt-get install -y php$PHP_74-fpm php$PHP_74-cli 
RUN apt-get install -y php$PHP_74-gd php$PHP_74-json php$PHP_74-mysql
RUN apt-get install -y php$PHP_74-readline php$PHP_74-mbstring
RUN apt-get install -y php$PHP_74-dom php$PHP_74-curl
RUN apt-get install -y php$PHP_74-zip php$PHP_74-redis php$PHP_74-memcached
RUN apt-get install -y php$PHP_74-bcmath php$PHP_74-bz2 php$PHP_74-gmp php$PHP_74-mongo php$PHP_74-intl php$PHP_74-soap php$PHP_74-amqp

# Install Xdebug
RUN apt-get install -y php${PHP_VERSION}-xdebug php${PHP_74}-xdebug

# Add Xdebug to PHP configuration for xdebug 3
RUN echo "" >> /etc/php/$PHP_VERSION/mods-available/xdebug.ini \
 && echo "xdebug.mode=debug,develop" >> /etc/php/$PHP_VERSION/mods-available/xdebug.ini \
 && echo "xdebug.start_with_request=yes" >> /etc/php/$PHP_VERSION/mods-available/xdebug.ini \
 #&& echo "xdebug.client_host=host.docker.internal" >> /etc/php/$PHP_VERSION/mods-available/xdebug.ini \
 && echo "xdebug.client_port=9013" >> /etc/php/$PHP_VERSION/mods-available/xdebug.ini

RUN echo "" >> /etc/php/$PHP_74/mods-available/xdebug.ini \
 && echo "xdebug.mode=debug,develop" >> /etc/php/$PHP_74/mods-available/xdebug.ini \
 && echo "xdebug.start_with_request=yes" >> /etc/php/$PHP_74/mods-available/xdebug.ini \
 #&& echo "xdebug.client_host=host.docker.internal" >> /etc/php/$PHP_74/mods-available/xdebug.ini \
 && echo "xdebug.client_port=9023" >> /etc/php/$PHP_74/mods-available/xdebug.ini

# Start services
RUN service php$PHP_VERSION-fpm start
RUN service php$PHP_74-fpm start
RUN service supervisor start
RUN service nginx start

RUN chown -R www-data:www-data /var/www

COPY config/etc/php/$PHP_VERSION/fpm/pool.d/www.conf /etc/php/$PHP_VERSION/fpm/pool.d/www.conf
COPY config/etc/php/$PHP_VERSION/fpm/php.ini /etc/php/$PHP_VERSION/fpm/php.ini
COPY config/etc/php/$PHP_VERSION/fpm/php-fpm.conf /etc/php/$PHP_VERSION/fpm/php-fpm.conf

#COPY config/etc/php/$PHP_74/fpm/pool.d/www.conf /etc/php/$PHP_74/fpm/pool.d/www.conf
#COPY config/etc/php/$PHP_74/fpm/php.ini /etc/php/$PHP_74/fpm/php.ini
#COPY config/etc/php/$PHP_74/fpm/php-fpm.conf /etc/php/$PHP_74/fpm/php-fpm.conf

#RUN apt-get upgrade -y && apt-get autoremove

# Install mongodb
#MAINTAINER dinhquocvietpro@gmail.com
#RUN apt-get install gnupg
#RUN wget -qO - https://www.mongodb.org/static/pgp/server-4.2.asc | apt-key add -
#RUN echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu bionic/mongodb-org/4.2 multiverse" | tee /etc/apt/sources.list.d/mongodb-org-4.2.list
#RUN apt-get update
#RUN apt-get install -y mongodb-org

# Install memcached
RUN apt-get -y install memcached

#### Local Only ####
# Copy Nginx Configs
# COPY config/etc/nginx/sites-available/ /etc/nginx/sites-available/

# Replace original sites-available by making a symlink to mapped sites-available dir
# make sure the sites-available on local machine is mapped into docker via docker-compose file
#backup original sites-available dir
RUN mv /etc/nginx/sites-available /etc/nginx/sites-available-bak
#make symlink to mapped sites-available dir
RUN ln -s /var/www/config/etc/nginx/sites-available /etc/nginx/sites-available
