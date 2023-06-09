# Set master image
FROM php:8.1-fpm

# Arguments defined in docker-compose.yml
ARG USER=murauer
ARG UID=1000
ENV USER ${USER}
ENV UID ${UID}
ARG NODEJS=16
ENV NODEJS ${NODEJS}

# Set working directory
WORKDIR /var/www

# Install system dependencies
RUN apt-get update && \
	apt-get install -y --no-install-recommends build-essential\
    sudo \
    openssl \
    make \
    vim \
    curl \
    npm \
    git \
	libpng-dev \
	libonig-dev \
	libxml2-dev \
	zip \
	unzip

# Clear cache
RUN apt-get clean && rm -rf /var/lib/apt/lists/*

# Install PHP extensions
RUN docker-php-ext-install pdo_mysql mbstring exif pcntl bcmath gd

# Install nodejs
RUN curl -sL https://deb.nodesource.com/setup_${NODEJS}.x | bash -
# @see check nodejs versions here https://github.com/sass/node-sass#node-version-support-policy
RUN apt-get update
RUN apt-get -y install build-essential nodejs

# Install xdebug
RUN pecl install xdebug \
    && docker-php-ext-enable xdebug

# Get latest Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# setup npm
RUN npm install -g npm@latest

# Expose port 9000 and start php-fpm server
EXPOSE 9000
CMD ["php-fpm"]

# Create system user to run Composer and Artisan Commands
RUN useradd -G www-data,root -u ${UID} -d /home/${USER} ${USER}
RUN mkdir -p /home/${USER}/.composer && \
    chown -R ${USER}:${USER} /home/${USER}

USER ${USER}
