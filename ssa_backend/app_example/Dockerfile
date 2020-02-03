FROM php:7.2-fpm
 
# Copiar composer.lock y composer.json
#COPY ssaysen/composer.json /var/www/


# Configura el directorio raiz
WORKDIR /var/www
 
# Instalamos dependencias
RUN apt-get update && apt-get install -y \
    build-essential \
    libpng-dev \
    libjpeg62-turbo-dev \
    libfreetype6-dev \
    locales \
    zip \
    jpegoptim optipng pngquant gifsicle \
    vim \
    unzip \
    git \
    curl \
    libpq-dev
 
# Borramos cache
RUN apt-get clean && rm -rf /var/lib/apt/lists/*
 
# Instalamos extensiones
RUN docker-php-ext-install pdo pdo_pgsql pdo_mysql mbstring zip exif pcntl
RUN docker-php-ext-configure gd --with-gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ --with-png-dir=/usr/include/
RUN docker-php-ext-install gd

# Instalar composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
 
# agregar usuario para la aplicación laravel
RUN groupadd -g 1000 www
RUN useradd -u 1000 -ms /bin/bash -g www www
 
# Copiar el directorio existente a /var/www
COPY ssaysen /var/www
 
# copiar los permisos del directorio de la aplicación
COPY --chown=www:www ssaysen /var/www
 
# cambiar el usuario actual por www
USER www

# install oci
COPY var/instantclient-basic-linux.x64-12.2.0.1.0.zip \
     var/instantclient-sdk-linux.x64-12.2.0.1.0.zip \
     var/instantclient-sqlplus-linux.x64-12.2.0.1.0.zip /tmp/

USER root

ENV LD_LIBRARY_PATH /usr/local/instantclient

RUN apt-get update && apt-get install -y unzip zip libaio-dev && unzip -o /tmp/instantclient-basic-linux.x64-12.2.0.1.0.zip -d /usr/local/ \
     && unzip -o /tmp/instantclient-sdk-linux.x64-12.2.0.1.0.zip -d /usr/local/ \
     && unzip -o /tmp/instantclient-sqlplus-linux.x64-12.2.0.1.0.zip -d /usr/local/ \
     && ln -s /usr/local/instantclient_12_2 /usr/local/instantclient \
     && ln -s /usr/local/instantclient/libclntsh.so.12.1 /usr/local/instantclient/libclntsh.so \
     && ln -s /usr/local/instantclient/sqlplus /usr/bin/sqlplus \
     && echo 'export LD_LIBRARY_PATH="/usr/local/instantclient"' >> /root/.bashrc \
     && echo 'export ORACLE_HOME="/usr/local/instantclient"' >> /root/.bashrc \
     && echo 'umask 002' >> /root/.bashrc \
     && docker-php-ext-configure oci8 -with-oci8=instantclient,/usr/local/instantclient \
&& docker-php-ext-install oci8 

RUN echo 'ULTIMO PASO'

USER www

# exponer el puerto 9000 e iniciar php-fpm server
EXPOSE 9000
CMD ["php-fpm"]
