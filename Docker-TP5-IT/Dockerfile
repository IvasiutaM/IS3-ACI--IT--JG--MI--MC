# Usar la imagen oficial de PHP con Apache
FROM php:8.2-apache

# Habilitar mod_rewrite para el ruteo amigable de CodeIgniter
RUN a2enmod rewrite

# Instalar dependencias del sistema y extensiones de PHP necesarias para CI4
RUN apt-get update && apt-get install -y \
    libicu-dev \
    libzip-dev \
    zip \
    unzip \
    && docker-php-ext-configure intl \
    && docker-php-ext-install intl mysqli pdo pdo_mysql zip

# Configuración CRUCIAL: Cambiar el DocumentRoot a la carpeta /public de CI4
ENV APACHE_DOCUMENT_ROOT /var/www/html/public
RUN sed -ri -e 's!/var/www/html!${APACHE_DOCUMENT_ROOT}!g' /etc/apache2/sites-available/*.conf
RUN sed -ri -e 's!/var/www/!${APACHE_DOCUMENT_ROOT}!g' /etc/apache2/apache2.conf /etc/apache2/conf-available/*.conf

# Establecer el directorio de trabajo
WORKDIR /var/www/html
