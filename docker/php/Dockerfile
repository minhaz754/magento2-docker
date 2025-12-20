FROM php:8.3-fpm


# System dependencies
RUN apt-get update && apt-get install -y \
libfreetype6-dev \
libjpeg62-turbo-dev \
libpng-dev \
libzip-dev \
libicu-dev \
libxslt1-dev \
zip unzip git curl \
&& docker-php-ext-configure gd --with-freetype --with-jpeg \
&& docker-php-ext-install -j$(nproc) \
bcmath
intl
mysqli
pdo_mysql
soap
sockets
xsl
zip
gd
opcache


# Composer
COPY --from=composer:2 /usr/bin/composer /usr/bin/composer


WORKDIR /var/www/html


RUN usermod -u 1000 www-data && groupmod -g 1000 www-data