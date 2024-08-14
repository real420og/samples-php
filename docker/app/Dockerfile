FROM php:8.3-cli

RUN apt-get update && apt-get install -y --no-install-recommends \
  nano \
  bash \
  libzip-dev \
  unzip \
  libonig-dev

# Install PHP Extensions
ENV CFLAGS="$CFLAGS -D_GNU_SOURCE"
RUN docker-php-ext-install zip \
  && docker-php-ext-install opcache sockets mbstring \
  && docker-php-ext-enable opcache sockets mbstring

# Protobuf and GRPC
ENV PROTOBUF_VERSION "3.25.1"
RUN pecl channel-update pecl.php.net
RUN pecl install protobuf-${PROTOBUF_VERSION} grpc \
    && docker-php-ext-enable protobuf grpc

COPY --from=mlocati/php-extension-installer /usr/bin/install-php-extensions /usr/local/bin/

ARG WITH_XDEBUG=0
RUN if [ $WITH_XDEBUG -eq "1" ]; then \
    install-php-extensions xdebug \
    && echo 'xdebug.mode=debug' >> "$PHP_INI_DIR/conf.d/docker-php-ext-xdebug.ini" \
    && echo 'xdebug.start_with_request=yes' >> "$PHP_INI_DIR/conf.d/docker-php-ext-xdebug.ini" \
    && echo 'xdebug.log_level=0' >> "$PHP_INI_DIR/conf.d/docker-php-ext-xdebug.ini"; \
  fi

# Install Temporal CLI
COPY --from=temporalio/admin-tools /usr/local/bin/tctl /usr/local/bin/tctl

# Install Composer
COPY --from=composer /usr/bin/composer /usr/local/bin/composer

# Wait for Temporal service to star up
COPY wait-for-temporal.sh /usr/local/bin
RUN chmod +x /usr/local/bin/wait-for-temporal.sh

# Copy application codebase
WORKDIR /var/app
COPY app/ /var/app

RUN composer install

# Setup RoadRunner
RUN vendor/bin/rr get --no-interaction \
    && mv rr /usr/local/bin/rr \
    && chmod +x /usr/local/bin/rr
