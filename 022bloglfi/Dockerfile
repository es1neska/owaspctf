FROM php:7.4-apache
run apt-get update && apt-get install nano -y
COPY src/ /var/www/html/
RUN chmod -R 777 /var/www/html/ViroCMS/
