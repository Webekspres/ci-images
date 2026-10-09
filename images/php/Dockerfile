# ci-php: PHP + composer + Node/Bun + klien MySQL/Postgres/Redis untuk proyek Laravel/Bagisto.
ARG PHP_VERSION=8.3
FROM docker.io/library/php:${PHP_VERSION}-cli-bookworm

COPY --from=docker.io/mlocati/php-extension-installer:2 /usr/bin/install-php-extensions /usr/local/bin/

RUN install-php-extensions \
        bcmath calendar exif gd gmp intl mysqli opcache pcntl pdo_mysql pdo_pgsql redis soap sockets zip \
        @composer \
    && echo "memory_limit=2G" > /usr/local/etc/php/conf.d/ci.ini

ARG NODE_MAJOR=22
RUN apt-get update -qq \
    && apt-get install -y -qq --no-install-recommends ca-certificates curl git gnupg unzip default-mysql-client postgresql-client redis-tools openssh-client rsync \
    && curl -fsSL https://deb.nodesource.com/setup_${NODE_MAJOR}.x | bash - >/dev/null \
    && apt-get install -y -qq --no-install-recommends nodejs \
    && npm install -g bun@1 >/dev/null 2>&1 \
    && rm -rf /var/lib/apt/lists/* /root/.npm

# checkout di dalam kontainer berjalan sebagai root pada folder milik user runner
RUN git config --system --add safe.directory '*'

ENV COMPOSER_ALLOW_SUPERUSER=1 COMPOSER_NO_INTERACTION=1
WORKDIR /workspace
