# =============================================================================
# Production Dockerfile for OpenKOS (Hardened against CVE-2026-109618 / 109619)
# =============================================================================

# --- Stage 1: Build Assets ---
FROM node:22-alpine AS frontend
WORKDIR /app
COPY package*.json pnpm-lock*.yaml* ./
RUN npm install
COPY . .
RUN npm run build

# --- Stage 2: Application Runner ---
FROM php:8.4-fpm-alpine

WORKDIR /var/www/html

# Install system dependencies and PHP extensions
RUN apk add --no-cache \
    postgresql-dev \
    libpng-dev \
    libzip-dev \
    zip \
    unzip \
    icu-dev \
    oniguruma-dev \
    curl \
    git \
    supervisor \
    nginx

RUN docker-php-ext-install \
    pdo \
    pdo_pgsql \
    pgsql \
    mbstring \
    zip \
    exif \
    pcntl \
    bcmath \
    intl \
    opcache

# Hardened php.ini settings for PHP2Shell mitigation
RUN echo "session.upload_progress.enabled = 0" > /usr/local/etc/php/conf.d/99-security.ini && \
    echo "session.upload_progress.cleanup = 1" >> /usr/local/etc/php/conf.d/99-security.ini && \
    echo "expose_php = Off" >> /usr/local/etc/php/conf.d/99-security.ini && \
    echo "memory_limit = 512M" >> /usr/local/etc/php/conf.d/99-security.ini && \
    echo "upload_max_filesize = 50M" >> /usr/local/etc/php/conf.d/99-security.ini && \
    echo "post_max_size = 50M" >> /usr/local/etc/php/conf.d/99-security.ini

# Get Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# Copy code
COPY . /var/www/html
COPY --from=frontend /app/public/build /var/www/html/public/build

# Set permissions
RUN chown -R www-data:www-data /var/www/html/storage /var/www/html/bootstrap/cache

EXPOSE 8080

CMD ["php-fpm"]
