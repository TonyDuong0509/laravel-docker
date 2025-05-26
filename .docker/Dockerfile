# 1. Base image PHP with FPM
FROM php:8.3-fpm

# 2. Install dependencies
RUN apt-get update && apt-get install -y \
    libfreetype6-dev \
    libjpeg62-turbo-dev \
    libpng-dev \
    libonig-dev \
    libzip-dev \
    libicu-dev \
    zip \
    unzip \
    git \
    curl \
    wget \
    supervisor \
    cron \
    nano \
    && docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install pdo pdo_mysql mbstring zip exif pcntl gd bcmath intl \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# 3. Set the working directory
WORKDIR /var/www/html

# 4. Install Composer
RUN wget https://getcomposer.org/installer -O composer-setup.php \
    && php composer-setup.php --install-dir=/usr/local/bin --filename=composer \
    && rm composer-setup.php
ENV COMPOSER_ALLOW_SUPERUSER=1

# 5. Đặt quyền cho các thư mục storage và cache & start script
COPY ../.docker/start_app.sh /usr/local/bin/start_app.sh
RUN chmod +x /usr/local/bin/start_app.sh

### Supervisord
COPY ../.docker/supervisord.conf /etc/supervisor/conf.d/worker.conf
RUN mkdir -p /var/log/supervisor
RUN touch /var/log/supervisor/supervisord.log
RUN chown -R www-data:www-data /var/log/supervisor

### Cronjob
COPY ../.docker/cronjob /etc/cron.d/laravel-cron
RUN chmod 0644 /etc/cron.d/laravel-cron
RUN crontab /etc/cron.d/laravel-cron
RUN touch /var/log/cron.log

# Install Node.js and npm
RUN curl -fsSL https://deb.nodesource.com/setup_current.x | bash - \
    && apt-get install -y nodejs \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# Expose port 9000 for PHP-FPM
EXPOSE 9000
CMD /usr/local/bin/start_app.sh
