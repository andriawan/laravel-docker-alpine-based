# reference from https://github.com/jabardigitalservice/docker-phpfpm-nginx
FROM alpine:3.10

ADD https://dl.bintray.com/php-alpine/key/php-alpine.rsa.pub /etc/apk/keys/php-alpine.rsa.pub

# make sure you can use HTTPS
RUN apk --update add ca-certificates

RUN echo "https://dl.bintray.com/php-alpine/v3.10/php-7.4" >> /etc/apk/repositories

# Install packages
RUN apk --no-cache add php musl php-fpm php-opcache php-openssl php-curl \
    nginx supervisor curl php-json php-phar php-iconv \
    php-exif php-sodium php-pdo php-mbstring php-dom \
    php-zip php-mysqli php-sqlite3 php-session php-bcmath \
    php-common php-gd php-intl php-fileinfo php-pdo_mysql

# https://github.com/codecasts/php-alpine/issues/21
RUN ln -s /usr/bin/php7 /usr/bin/php

# install composer

RUN curl -s https://getcomposer.org/installer | php
RUN mv composer.phar /usr/local/bin/composer

# Configure nginx
COPY docker/config/nginx.conf /etc/nginx/nginx.conf

# Remove default server definition
RUN rm /etc/nginx/conf.d/default.conf

# Configure PHP-FPM
COPY docker/config/fpm-pool.conf /etc/php7/php-fpm.d/www.conf
COPY docker/config/php.ini /etc/php7/conf.d/custom.ini

# Configure supervisord
COPY docker/config/supervisord.conf /etc/supervisor/conf.d/supervisord.conf

# Setup document root
RUN mkdir -p /var/www/website

# Add application
WORKDIR /var/www/website
COPY --chown=nobody . /var/www/website/
#COPY .env.example .env

RUN composer install --prefer-dist --no-ansi --no-interaction --no-progress --no-scripts

RUN chmod -R 755 /var/www/website/storage
RUN chmod -R 755 /var/www/website/public
RUN php artisan key:generate
RUN php artisan config:clear
RUN php artisan cache:clear
RUN php artisan view:clear
RUN php artisan route:clear

RUN composer dump-autoload --optimize --classmap-authoritative

RUN chown -R nobody.nobody /var/www/website && \
  chown -R nobody.nobody /run && \
  chown -R nobody.nobody /var/lib/nginx && \
  chown -R nobody.nobody /var/log/nginx

# Switch to use a non-root user from here on
USER nobody
# Expose the port nginx is reachable on
EXPOSE 8080

# Let supervisord start nginx & php-fpm
CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]

# Configure a healthcheck to validate that everything is up&running
HEALTHCHECK --timeout=10s CMD curl --silent --fail http://127.0.0.1:8080/fpm-ping
