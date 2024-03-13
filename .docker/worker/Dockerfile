#base-WORKER-images Amazon Elastic Container Registry
FROM php:8.2.2-cli-alpine

ARG SERVER_ENVIRONMENT
ARG APP_DIR=/var/www/app
WORKDIR $APP_DIR
# Add Repositories
RUN rm -f /etc/apk/repositories &&\
    echo "http://dl-cdn.alpinelinux.org/alpine/v3.15/main" >> /etc/apk/repositories && \
    echo "http://dl-cdn.alpinelinux.org/alpine/v3.15/community" >> /etc/apk/repositories

# Add Build Dependencies
RUN apk update && apk add --no-cache  \
    libzip-dev \
    libmcrypt-dev \
    libjpeg-turbo-dev \
    libjpeg-turbo \
    jpeg-dev \
    libpng-dev \
    libxml2-dev \
    bzip2-dev \
    libwebp-dev \
    zip \
    jpegoptim \
    pngquant \
    optipng \
    icu-dev \
    freetype-dev \
    libpq \
    zlib-dev \
    imap-dev \
    git \
    vim \
    openssl-dev \
    pcre-dev  \
    pcre2-dev \
    supervisor


RUN set -ex \
  && apk --no-cache add \
    postgresql-dev

# Configure & Install Extension
RUN docker-php-ext-configure pgsql -with-pgsql=/usr/local/pgsql && \
    docker-php-ext-install -j "$(nproc)" \
    opcache \
    pdo \
    pdo_pgsql \
    pgsql


RUN apk add --update --no-cache autoconf g++ make \
&& pecl install redis \
&&  docker-php-ext-enable redis


RUN docker-php-ext-configure gd --with-freetype --with-jpeg --with-webp && \
    docker-php-ext-install -j$(nproc) gd

RUN docker-php-ext-configure pcntl --enable-pcntl \
  && docker-php-ext-install pcntl

#RUN pecl install openswoole-4.7.2 && docker-php-ext-enable openswoole
# Crontab
RUN apk update
RUN touch /var/log/cron.log
COPY .docker/worker/cron /etc/cron.d/cron
RUN chmod -R 755 /etc/cron.d/cron
RUN /usr/bin/crontab /etc/cron.d/cron

# Supervisor & Copy app
RUN cd $APP_DIR
RUN chown www-data:www-data $APP_DIR
#COPY --chown=www-data:www-data . .
COPY .docker/worker/worker.conf /etc/supervisor/conf.d/worker.conf
COPY .docker/worker/supervisord.conf /etc/supervisord.conf
RUN chmod -R 644 /etc/supervisor/conf.d/worker.conf
RUN chmod -R 644 /etc/supervisord.conf
RUN touch /tmp/supervisor.sock
RUN chmod 777 /tmp/supervisor.sock

COPY .docker/worker/opcache.ini $PHP_INI_DIR/conf.d/
COPY .docker/worker/php.ini $PHP_INI_DIR/conf.d/
RUN mkdir -p /var/www/app/bootstrap/cache
RUN chmod -R 775 /var/www/app/bootstrap/cache
RUN curl -sS https://getcomposer.org/installer | \
php -- --install-dir=/usr/bin/ --filename=composer
#RUN composer install  --no-interaction --optimize-autoloader --no-dev --prefer-dist
#RUN chown -R www-data:www-data storage

ENV COMPOSER_ALLOW_SUPERUSER=1

CMD ["/usr/bin/supervisord", "-c", "/etc/supervisord.conf"]