# Use a more recent PHP version with Debian buster as the base
FROM php:7.4-apache

# Install mysqli extension
RUN docker-php-ext-install mysqli

# Copy application files and set permissions
COPY src/ /var/www/html/
RUN chmod -R 755 /var/www/html/