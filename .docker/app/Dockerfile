FROM php:8.4-fpm-alpine

ARG SERVER_ENVIRONMENT
ARG APP_DIR=/var/www/app
# Setup Working Dir
WORKDIR $APP_DIR
RUN set -ex \
  && apk --no-cache add \
    postgresql-dev

# Add Repositories
RUN rm -f /etc/apk/repositories &&\
    echo "http://dl-cdn.alpinelinux.org/alpine/v3.18/main" >> /etc/apk/repositories && \
    echo "http://dl-cdn.alpinelinux.org/alpine/v3.18/community" >> /etc/apk/repositories

# fix work iconv library with alpine
RUN apk add --no-cache --repository http://dl-cdn.alpinelinux.org/alpine/edge/community/ --allow-untrusted gnu-libiconv
ENV LD_PRELOAD /usr/lib/preloadable_libiconv.so php

# Add Build Dependencies
RUN apk add --no-cache \
    php82-pear \
    libwebp-dev \
    libzip-dev \
    libjpeg-turbo-dev \
    libjpeg-turbo \
    libpng-dev \
    libxpm-dev \
    php82-dev gcc \
    zlib-dev \
    curl-dev \
    imagemagick \
    imagemagick-dev \
    freetype-dev \
    icu-dev \
    g++ \
    npm \
    zip

# Configure & Install Extension
RUN docker-php-ext-install -j "$(nproc)" \
    curl \
    pgsql \
    pdo \
    pdo_pgsql \
    bcmath \
    zip


RUN docker-php-ext-configure intl && \
    docker-php-ext-install intl &&\
    docker-php-ext-configure pgsql -with-pgsql=/usr/local/pgsql


RUN docker-php-ext-configure gd --with-freetype --with-jpeg --with-webp && \
    docker-php-ext-install -j$(nproc) gd

RUN docker-php-ext-configure pcntl --enable-pcntl \
  && docker-php-ext-install pcntl


# Install Build Tools
RUN apk add --no-cache build-base

# Install Imagick PHP extension
RUN apk update && apk upgrade

RUN pecl install imagick && docker-php-ext-enable imagick

# RUN apk add --no-cache --virtual .build-deps $PHPIZE_DEPS imagemagick-dev \
#     && pecl install imagick \
#     && docker-php-ext-enable imagick \
#     && apk del .build-deps


# Configure and enable OPCache extension
RUN docker-php-ext-configure opcache --enable-opcache
RUN docker-php-ext-install opcache

# Add Node.js v14.17.6 or other 14.x version
#RUN curl -sL https://unofficial-builds.nodejs.org/download/release/v14.0.0/node-v14.0.0-linux-x64-musl.tar.gz  | tar xz -C /usr/local --strip-components=1

# Add Composer lastest version
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/bin/ --filename=composer

# Install Redis
RUN pecl install redis && docker-php-ext-enable redis
RUN apk add --no-cache postgresql-client


RUN cd $APP_DIR
RUN chown www-data:www-data $APP_DIR
#COPY --chown=www-data:www-data . .
RUN mkdir -p /var/www/app/bootstrap/cache
RUN chmod -R 775 /var/www/app/bootstrap/cache
RUN curl -sS https://getcomposer.org/installer | \
php -- --install-dir=/usr/bin/ --filename=composer
#RUN composer install  --no-interaction --optimize-autoloader --no-dev --prefer-dist
#RUN chown -R www-data:www-data storage
COPY .docker/app/opcache.ini $PHP_INI_DIR/conf.d/
COPY .docker/app/php.ini $PHP_INI_DIR/conf.d/


COPY .docker/app/init.sh /usr/local/bin
RUN chmod +x /usr/local/bin/init.sh

ENTRYPOINT ["/usr/local/bin/init.sh"]