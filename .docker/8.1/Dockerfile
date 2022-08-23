FROM php:8.1-apache

WORKDIR /var/www/html

ENV DEBIAN_FRONTEND noninteractive
ENV TZ=UTC

ARG WWWGROUP
ARG WWWUSER
RUN groupadd --force -g $WWWGROUP $WWWUSER
RUN useradd -ms /bin/bash --no-user-group -g $WWWGROUP -u 1337 $WWWUSER

# Setup locale
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# Install packages
RUN mkdir -p ~/.gnupg \
    && chmod 600 ~/.gnupg \
    && echo "disable-ipv6" >> ~/.gnupg/dirmngr.conf \
    && php -r "readfile('http://getcomposer.org/installer');" | php -- --install-dir=/usr/bin/ --filename=composer \
    && apt update \
    && apt install -y --no-install-recommends locales net-tools libbz2-dev libzip-dev \
    && apt dist-upgrade -y \
    && apt -y autoremove \
    && apt clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Setup locale
RUN sed -i -e 's/# en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/' /etc/locale.gen \
    && dpkg-reconfigure --frontend=noninteractive locales \
    && update-locale LANG=en_US.UTF-8
ENV LANG en_US.UTF-8
ENV LC_ALL en_US.UTF-8

# Install PHP Extensions
RUN docker-php-ext-install \
    bz2 \
    zip \
    pdo_mysql

# Configure apache2
RUN a2enmod rewrite

# Add some usefull aliases
RUN echo 'alias a="php artisan"' >> ~/.bashrc \
    && echo 'alias p="/var/www/api/vendor/bin/phpunit"' >> ~/.bashrc

# Configure php
RUN cp /usr/local/etc/php/php.ini-production /usr/local/etc/php/php.ini \
    && echo "date.timezone=Europe/Paris" >> /usr/local/etc/php/php.ini
COPY .docker/8.1/php.ini /etc/php/8.1/cli/conf.d/99-docker.ini
COPY .docker/8.1/000-default.conf /etc/apache2/sites-available/000-default.conf

# Run composer
COPY composer.json composer.lock ./
RUN composer install --prefer-dist --no-dev --no-interaction --no-autoloader --no-scripts

# Copy code
COPY . /var/www/html/

# Finalize
RUN cp .env.example .env \
    && composer dump-autoload --optimize \
    && chmod -R 777 public

# Manage user, config apache & final prep
ARG WWWGROUP
ARG WWWUSER
ARG UID
RUN groupadd --force -g $WWWGROUP sail \
    && useradd -ms /bin/bash --no-user-group -g $WWWGROUP -u $WWWUSER sail \
    && echo "Listen 5000" >> /etc/apache2/ports.conf

ENV APACHE_RUN_USER=sail
ENV APACHE_RUN_GROUP=sail
# Ensure PHP logs are captured by the container
ENV LOG_CHANNEL=stderr
ENV XDEBUG_MODE=coverage

EXPOSE 5000
