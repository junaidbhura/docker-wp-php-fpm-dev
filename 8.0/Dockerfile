# https://github.com/docker-library/wordpress/blob/master/latest/php8.0/fpm/Dockerfile
FROM php:8.0-fpm

# persistent dependencies
RUN set -eux; \
	apt-get update; \
	apt-get install -y --no-install-recommends \
# Ghostscript is required for rendering PDF previews
		ghostscript \
	; \
	rm -rf /var/lib/apt/lists/*

# PHP Extensions
RUN set -ex; \
	\
	savedAptMark="$(apt-mark showmanual)"; \
	\
	apt-get update; \
	apt-get install -y --no-install-recommends \
		libfreetype6-dev \
		libjpeg-dev \
		libmagickwand-dev \
		libpng-dev \
		libzip-dev \
	; \
	\
	docker-php-ext-configure gd \
		--with-freetype \
		--with-jpeg \
	; \
	docker-php-ext-install -j "$(nproc)" \
		bcmath \
		exif \
		gd \
		mysqli \
		zip \
	; \
# https://pecl.php.net/package/imagick
	pecl install imagick-3.5.0; \
	docker-php-ext-enable imagick; \
	rm -r /tmp/pear; \
	\
# reset apt-mark's "manual" list so that "purge --auto-remove" will remove all build dependencies
	apt-mark auto '.*' > /dev/null; \
	apt-mark manual $savedAptMark; \
	ldd "$(php -r 'echo ini_get("extension_dir");')"/*.so \
		| awk '/=>/ { print $3 }' \
		| sort -u \
		| xargs -r dpkg-query -S \
		| cut -d: -f1 \
		| sort -u \
		| xargs -rt apt-mark manual; \
	\
	apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false; \
	rm -rf /var/lib/apt/lists/*

# OPCache
RUN set -eux; \
	docker-php-ext-enable opcache; \
	{ \
		echo 'opcache.memory_consumption=128'; \
		echo 'opcache.interned_strings_buffer=8'; \
		echo 'opcache.max_accelerated_files=4000'; \
		echo 'opcache.revalidate_freq=2'; \
		echo 'opcache.fast_shutdown=1'; \
	} > /usr/local/etc/php/conf.d/opcache-recommended.ini

# Error Constants
RUN { \
    echo 'error_reporting = E_ERROR | E_WARNING | E_PARSE | E_CORE_ERROR | E_CORE_WARNING | E_COMPILE_ERROR | E_COMPILE_WARNING | E_RECOVERABLE_ERROR'; \
    echo 'display_errors = Off'; \
    echo 'display_startup_errors = Off'; \
    echo 'log_errors = On'; \
    echo 'error_log = /dev/stderr'; \
    echo 'log_errors_max_len = 1024'; \
    echo 'ignore_repeated_errors = On'; \
    echo 'ignore_repeated_source = Off'; \
    echo 'html_errors = Off'; \
} > /usr/local/etc/php/conf.d/error-logging.ini

# Composer
RUN curl -sS https://getcomposer.org/installer | php \
	&& mv composer.phar /usr/local/bin/composer \
	&& apt-get update && apt-get install -y \
		zlib1g-dev \
        libzip-dev \
        unzip \
    && docker-php-ext-install zip

# WP CLI
RUN curl -O https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar \
	&& chmod +x wp-cli.phar \
	&& mv wp-cli.phar /usr/local/bin/wp

# Install less, required for wp-cli.
RUN apt-get update && apt-get install -y less

# XDebug
RUN pecl install xdebug \
    && docker-php-ext-enable xdebug \
    && { \
        echo 'zend_extension=xdebug'; \
        echo '[xdebug]'; \
        echo 'xdebug.mode=off'; \
        echo 'xdebug.start_with_request=yes'; \
        echo 'xdebug.client_host=host.docker.internal'; \
    } > /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini

# Memcache and Memcached Extensions
RUN pecl install memcache \
    && docker-php-ext-enable memcache

RUN apt-get install -y libz-dev libmemcached-dev \
    && pecl install memcached \
    && docker-php-ext-enable memcached

# Redis Extension
RUN pecl install redis-5.3.4 \
    && docker-php-ext-enable redis

# MailHog
RUN apt-get install -y wget \
    && wget https://github.com/mailhog/mhsendmail/releases/download/v0.2.0/mhsendmail_linux_amd64 \
    && chmod +x mhsendmail_linux_amd64 \
    && mv mhsendmail_linux_amd64 /usr/local/bin/mhsendmail \
    && echo 'sendmail_path = "/usr/local/bin/mhsendmail --smtp-addr=mailhog:1025"' >> /usr/local/etc/php/conf.d/sendmail.ini

# Start
EXPOSE 9000
CMD ["php-fpm"]
