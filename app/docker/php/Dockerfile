FROM php:8.4-cli-alpine

RUN apk add --no-cache git unzip icu-dev libzip-dev \
 && docker-php-ext-install pdo_mysql intl zip pcntl

COPY --from=composer:2 /usr/bin/composer /usr/bin/composer

# Run as the host user, so files created in the container (migrations, models, vendor/)
# belong to you on the host. No more `sudo chown -R`.
ARG UID=1000
ARG GID=1000
RUN addgroup -g "${GID}" app && adduser -D -u "${UID}" -G app app
USER app

WORKDIR /var/www/html

# PHP's built-in server keeps running even before `composer install`, so `up --wait` can succeed on a fresh clone.
CMD ["php", "-S", "0.0.0.0:8000", "-t", "public"]
