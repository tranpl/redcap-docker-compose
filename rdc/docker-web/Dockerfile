#FROM php:8.2-apache-buster
FROM php:8.3-apache-bookworm

## https://hub.docker.com/_/php/tags?page=1&name=apache

ARG DEBIAN_FRONTEND=noninteractive

RUN apt-get update -qq && \
    apt-get -yq --no-install-recommends install \
    msmtp-mta \
    ca-certificates \
    git \
    vim \
    ssh \
    wget \
    libpng-dev \
    libzip-dev \
    zip \
    unzip \
    p7zip-full \
    ghostscript \
    libmagickwand-dev

# Install OpenLDAP development libraries
RUN apt-get update -qq && \
    apt-get -yq --no-install-recommends install \
    libldap2-dev \
    && docker-php-ext-install ldap \
    && docker-php-ext-enable ldap \
    && apt-get clean 

# INSTALL IMAGICK
# RUN pecl install imagick-3.7.0
# imagick: Install error "Unterminated preprocessor conditions" in php 8.3
# https://github.com/Imagick/imagick/issues/640
RUN curl -fsSL https://github.com/Imagick/imagick/archive/944b67fce68bcb5835999a149f917670555b6fcb.tar.gz | tar xvz -C /tmp \
    && yes '' | pecl install /tmp/imagick-944b67fce68bcb5835999a149f917670555b6fcb/package.xml \
    && docker-php-ext-enable imagick

    # Update ImageMagick policy
RUN sed -i 's/policy\ domain=\"coder\" rights=\"none\" pattern=\"PDF\"/policy domain=\"coder\" rights=\"read\" pattern=\"PDF\"/g' /etc/ImageMagick-6/policy.xml

# INSTALL MYSQLI AND OTHER DOCKER FUN (added freetype support for Gunther's nedcaptcha)
RUN docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install -j$(nproc) gd zip mysqli \

# INSTALL XDEBUG
    && pecl install xdebug \
    && echo "zend_extension=$(find /usr/local/lib/php/extensions/ -name xdebug.so)" > "/usr/local/etc/php/conf.d/xdebug.ini" \
### cleanup
    && rm -r /var/lib/apt/lists/*

# COPY OUR DEFAULT CONTAINER CONFIG
COPY ./container-config /etc/container-config
# THESE CONFIG FILES CAN BE SUPPLEMENTED OR OVERWRITTEN BY THE container_start.sh SCRIPT
# BY MOUNTING ADDITIONAL FILES INTO /etc/container-config-overwrite FOLDER.  THIS FOLDER
# IS ALIASED AS THE `WEB_OVERRIDES` ENV VARIABLE IN THE .env FILE

ENV SERVER_NAME=localhost
ENV SERVER_ADMIN=root
ENV SERVER_ALIAS=localhost
ENV APACHE_RUN_HOME=/var/www
ENV APACHE_DOCUMENT_ROOT=/var/www/html
ENV APACHE_ERROR_LOG=/dev/stdout
ENV APACHE_ACCESS_LOG=/dev/stdout

# If you are overriding this container, you might want to remove the default vhost
ENV REMOVE_DEFAULT_VHOST=false

# Copy main startup script
COPY container_start.sh /etc/container_start.sh

RUN chmod +x /etc/container_start.sh
CMD ["/etc/container_start.sh"]
