FROM php:8.4-cli-alpine

RUN apk add --no-cache git unzip icu-dev libzip-dev \
 && docker-php-ext-install pdo_mysql intl zip pcntl

COPY --from=composer:2 /usr/bin/composer /usr/bin/composer

WORKDIR /var/www/html

CMD ["php", "artisan", "serve", "--host=0.0.0.0", "--port=8000"]
