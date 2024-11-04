# Use a more recent PHP version with Debian buster as the base
FROM php:7.4-apache

# Update package list and install dependencies
RUN apt-get update && \
    apt-get install -y libfontconfig nano && \
    rm -rf /var/lib/apt/lists/*

# Copy application files and PHP configuration
COPY src/ /var/www/html/
COPY config/php.ini /usr/local/etc/php/

# Set ownership and permissions for application files
RUN chown root /var/www/html/* && \
    chmod 655 /var/www/html/* && \
    chmod 777 /var/www/html/images/