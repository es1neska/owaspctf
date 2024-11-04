FROM php:7.4-apache

# Install necessary dependencies and PHP extensions
RUN apt-get update && apt-get install -y \
    libfontconfig \
    nano \
    libfreetype6-dev \
    libjpeg62-turbo-dev \
    libpng-dev \
    && docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install -j$(nproc) iconv gd \
    && rm -rf /var/lib/apt/lists/*

# Copy application files and PHP configuration
COPY src/ /var/www/html/
COPY config/php.ini /usr/local/etc/php/
