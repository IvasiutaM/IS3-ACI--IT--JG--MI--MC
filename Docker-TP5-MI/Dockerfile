# ============================================================
# Dockerfile — Laravel (PHP 8.2 + extensiones necesarias)
# Proyecto: Sistema de Gestión de Eventos Académicos
# ============================================================

FROM php:8.2-fpm

# ── Argumentos de build ──────────────────────────────────────
ARG USER_ID=1000
ARG GROUP_ID=1000

# ── Dependencias del sistema ─────────────────────────────────
RUN apt-get update && apt-get install -y \
    git \
    curl \
    libpng-dev \
    libonig-dev \
    libxml2-dev \
    libpq-dev \
    libzip-dev \
    zip \
    unzip \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# ── Extensiones de PHP ───────────────────────────────────────
RUN docker-php-ext-install \
    pdo \
    pdo_pgsql \
    pgsql \
    mbstring \
    exif \
    pcntl \
    bcmath \
    gd \
    zip \
    xml

# ── Redis (extensión PECL) ───────────────────────────────────
RUN pecl install redis \
    && docker-php-ext-enable redis

# ── Composer ────────────────────────────────────────────────
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# ── Usuario no-root (evita problemas de permisos) ────────────
RUN groupadd -g ${GROUP_ID} laravel \
    && useradd -u ${USER_ID} -g laravel -m laravel \
    && mkdir -p /var/www/html \
    && chown -R laravel:laravel /var/www/html

USER laravel

# ── Directorio de trabajo ────────────────────────────────────
WORKDIR /var/www/html

# ── Exponer puerto PHP-FPM ───────────────────────────────────
EXPOSE 9000

CMD ["php-fpm"]
