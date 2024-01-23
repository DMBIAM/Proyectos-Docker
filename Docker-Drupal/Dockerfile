FROM amazonlinux:2

# ARG
ARG XDEBUG_VERSION
ARG DRUSH_VERSION
ARG PHP_VERSION

#USER PARAMS
USER root

# INSTALL NGINX
RUN yum -y update
COPY ./conf/nginx/nginx.repo /etc/yum.repos.d/
RUN yum -y update && yum -y install nginx && yum -y install initscripts

# INSTALL PING COMMAND
RUN yum install iputils

# INSTALL AND CONFIGURE PHP
RUN yum install gcc-c++ zlib-devel amazon-linux-extras -y
RUN amazon-linux-extras enable $PHP_VERSION && amazon-linux-extras install $PHP_VERSION -y
RUN yum -y install php php-bcmath php-cli php-mbstring php-xml php-opcache php-fpm php-intl php-gd php-pear php-devel gd gd-devel wget unzip memcached tar

# INSTALL XDEBUG
RUN pecl install $XDEBUG_VERSION 

# INSTALL AND CONFIGURE GIT
RUN yum -y install git  

# INSTALL AND CONFIGURE COMPOSER
RUN php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"   \
    && php composer-setup.php --install-dir=/usr/local/bin --filename=composer  \
    && php -r "unlink('composer-setup.php');"                                   \
    && ln -s /usr/local/bin/composer /usr/local/bin/composer.phar               \
    && composer self-update                                               \
    && composer --version -y

# INSTALL AND CONFIGURE DRUSH
RUN git clone --branch $DRUSH_VERSION https://github.com/drush-ops/drush.git /usr/local/src/drush \
    && cd /usr/local/src/drush \
    && ln -s /usr/local/src/drush/drush /usr/bin/drush
RUN cd /usr/local/src/drush && composer install -o -vvv

# SEPTUP SSH WITH SECURE ROOT LOGIN
RUN yum -y update \
    && yum -y install openssh-server netcat \
    && mkdir /var/run/sshd \
    && echo 'root:password' | chpasswd \
    && sed -i 's/\#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config

# CHANGE HOSTS FILE
COPY ./conf/hosts/hosts /etc/hosts

# LOAD CONFIG FOR NGINX
COPY ./conf/nginx/default.conf /etc/nginx/conf.d/default.conf

# LOAD PHP.INI CUSTOM
COPY ./conf/php/php.ini /etc/php.ini

# INSTALL SSL SELF SIGNED CERTIFICATE FOR DOMAIN TBO
RUN openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout nginx-selfsigned.key -out nginx-selfsigned.crt -subj "/C=CO/ST=BOG/L=BOG/O=Bits Americas, Inc./OU=IT/CN=local.tbo"

# RESTART NGIX AND PHP-FPM
RUN systemctl enable nginx.service && systemctl enable php-fpm.service

RUN export COMPOSER_PROCESS_TIMEOUT=6000
# RUN composer install -o -vvv

CMD ["/usr/sbin/init"] 