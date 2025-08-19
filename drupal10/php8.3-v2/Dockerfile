# Optimized Multi-Stage Dockerfile for Open Social
# Demonstrating various size reduction techniques

# Stage 1: Base with minimal runtime dependencies
FROM drupal:10.3.11-php8.3 AS base
LABEL org.opencontainers.image.authors="devel@goalgorilla.com"

# Install only essential runtime packages
RUN apt-get update && apt-get install -y \
    mariadb-client \
    msmtp \
    libzip-dev \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Install only essential PHP extensions
RUN curl -sSLf \
        -o /usr/local/bin/install-php-extensions \
        https://github.com/mlocati/docker-php-extension-installer/releases/latest/download/install-php-extensions \
    && chmod +x /usr/local/bin/install-php-extensions \
    && install-php-extensions zip bcmath intl redis

# Install Composer
RUN php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" \
    && php -r "if (hash_file('sha384', 'composer-setup.php') === 'dac665fdc30fdd8ec78b38b9800061b4150413ff2e3b6f88543c636f7cd84f6db9189d43a81e5503cda447da73c7e5b6') { echo 'Installer verified'.PHP_EOL; } else { echo 'Installer corrupt'.PHP_EOL; unlink('composer-setup.php'); exit(1); }" \
    && php composer-setup.php --install-dir=/usr/local/bin --filename=composer \
    && php -r "unlink('composer-setup.php');"

# Configure mail settings
COPY mailcatcher-msmtp.conf /etc/msmtprc
RUN echo 'sendmail_path = "/usr/bin/msmtp -t"' > /usr/local/etc/php/conf.d/mail.ini

# Stage 2: Dependencies installation (cached separately)
FROM base AS dependencies
WORKDIR /var/www

# Install git for dependency installation
RUN apt-get update && apt-get install -y git \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

COPY composer.json ./
ENV COMPOSER_ALLOW_SUPERUSER=1
ENV COMPOSER_MEMORY_LIMIT=-1
RUN composer install --prefer-dist --no-interaction --no-dev --optimize-autoloader --no-scripts

# Stage 3: Production environment (minimal)
FROM base AS production
WORKDIR /var/www/html

# Copy only production dependencies
COPY --from=dependencies /var/www/vendor /var/www/vendor

# Copy PHP configuration optimized for production
COPY php.ini /usr/local/etc/php/php.ini

# Set proper permissions
RUN chown -R www-data:www-data * \
    && php -r 'opcache_reset();'

# Stage 4: Development environment (includes dev tools)
FROM base AS development
WORKDIR /var/www

# Install additional development packages
RUN apt-get update && apt-get install -y \
    git curl wget nano vim 7zip \
    zlib1g-dev libssl-dev libxml2-dev \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Install additional PHP extensions for development
RUN install-php-extensions exif sockets soap gmp rdkafka

# Copy dependencies from dependencies stage
COPY --from=dependencies /var/www/vendor /var/www/vendor

# Copy composer files and install dev dependencies
COPY composer.json ./
ENV COMPOSER_ALLOW_SUPERUSER=1
ENV COMPOSER_MEMORY_LIMIT=-1
RUN composer install --prefer-dist --no-interaction

# Install development tools
RUN ln -s /root/.composer/vendor/bin/behat /usr/local/bin/behat \
    && ln -s /root/.composer/vendor/bin/phpunit /usr/local/bin/phpunit

# Set PATH for development tools
ENV PATH="$PATH:/var/www/vendor/bin:/usr/local/bin"

# Install and configure Xdebug for development
RUN pecl install xdebug-3.3.2 \
    && docker-php-ext-enable xdebug \
    && echo "xdebug.mode=debug" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
    && echo "xdebug.client_port=9003" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
    && echo "xdebug.discover_client_host=true" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
    && echo "xdebug.client_host=host.docker.internal" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
    && echo "xdebug.idekey=PHPSTORM" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
    && echo "xdebug.max_nesting_level=500" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini

# Configure shell
RUN echo "export TERM=xterm" >> ~/.bashrc

# Set proper permissions
WORKDIR /var/www/html
RUN chown -R www-data:www-data * \
    && php -r 'opcache_reset();'

# Stage 5: CI environment (testing focused)
FROM base AS ci
WORKDIR /var/www

# Install testing-specific packages
RUN apt-get update && apt-get install -y \
    git curl wget \
    zlib1g-dev libssl-dev libxml2-dev \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Install testing PHP extensions
RUN install-php-extensions exif sockets soap gmp rdkafka

# Copy dependencies from dependencies stage
COPY --from=dependencies /var/www/vendor /var/www/vendor

# Copy composer files and install all dependencies (including dev)
COPY composer.json ./
ENV COMPOSER_ALLOW_SUPERUSER=1
ENV COMPOSER_MEMORY_LIMIT=-1
RUN composer install --prefer-dist --no-interaction

# Install CI tools
RUN ln -s /root/.composer/vendor/bin/behat /usr/local/bin/behat \
    && ln -s /root/.composer/vendor/bin/phpunit /usr/local/bin/phpunit

# Set PATH for CI tools
ENV PATH="$PATH:/var/www/vendor/bin:/usr/local/bin"

# Default to production stage (smallest)
FROM production
