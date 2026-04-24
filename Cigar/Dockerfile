FROM php:7.4-apache
RUN sed -i 's!/var/www/html!/app!g' /etc/apache2/sites-available/000-default.conf
ADD www /app
