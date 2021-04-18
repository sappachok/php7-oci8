FROM php:7.2-apache

RUN apt-get update && apt-get install -y \
        unzip \
        libfreetype6-dev \
        libjpeg62-turbo-dev \
        libpng-dev \
        libaio1 \
	libfreetype6-dev \
        libssl-dev \
        libmcrypt-dev \
        zlib1g-dev \
        libxml2-dev \
        libonig-dev \
        zip \
        curl \
        libmemcached-dev \
        build-essential \
        libaio1 \
        libzip-dev \
    && docker-php-ext-install -j$(nproc) iconv gettext \
    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-install -j$(nproc) gd

COPY docker-php.conf /etc/apache2/conf-enabled/docker-php.conf

RUN printf "log_errors = On \nerror_log = /dev/stderr\n" > /usr/local/etc/php/conf.d/php-logs.ini

# Fix warnings
RUN echo "ServerName localhost" >> /etc/apache2/conf-available/localhost.conf && a2enconf localhost \

# Create self-signed SSL certificate
			&& mkdir /etc/apache2/ssl \
			&& openssl req \
				-x509 \
				-nodes \
				-days 365 \
				-newkey rsa:2048 \
				-keyout /etc/apache2/ssl/apache2.key \
				-out /etc/apache2/ssl/apache2.crt \
				-subj "/CN=localhost" \

# Remove packages to reduce image size
#			&& apt-get purge openssl \
#			&& apt-get autoremove --purge \

# Install SSL certificate
			&& sed -i -e "s|/etc/ssl/certs/ssl-cert-snakeoil.pem|/etc/apache2/ssl/apache2.crt|g" /etc/apache2/sites-available/default-ssl.conf \
			&& sed -i -e "s|/etc/ssl/private/ssl-cert-snakeoil.key|/etc/apache2/ssl/apache2.key|g" /etc/apache2/sites-available/default-ssl.conf

RUN a2enmod ssl
RUN a2ensite default-ssl

RUN a2enmod rewrite

# RUN apt-get update
# RUN apt-get install -y php-ldap


###########################################################################
# LDAP:
###########################################################################

ARG INSTALL_LDAP=true

RUN if [ ${INSTALL_LDAP} = true ]; then \
    apt-get install -y libldap2-dev && \
    docker-php-ext-configure ldap --with-libdir=lib/x86_64-linux-gnu/ && \
    docker-php-ext-install ldap \
;fi

# RUN a2enmod ldap

# Oracle instantclient
ADD ./instantclient/12.2.0.1.0/instantclient-basic-linux.x64-12.2.0.1.0.zip /tmp/
ADD ./instantclient/12.2.0.1.0/instantclient-sdk-linux.x64-12.2.0.1.0.zip /tmp/
ADD ./instantclient/12.2.0.1.0/instantclient-sqlplus-linux.x64-12.2.0.1.0.zip /tmp/

RUN unzip /tmp/instantclient-basic-linux.x64-12.2.0.1.0.zip -d /usr/local/
RUN unzip /tmp/instantclient-sdk-linux.x64-12.2.0.1.0.zip -d /usr/local/
RUN unzip /tmp/instantclient-sqlplus-linux.x64-12.2.0.1.0.zip -d /usr/local/

RUN ln -s /usr/local/instantclient_12_2 /usr/local/instantclient
RUN ln -s /usr/local/instantclient_12_2/libclntsh.so.12.1 /usr/local/instantclient/libclntsh.so
RUN ln -s /usr/local/instantclient_12_2/libocci.so.12.1 /usr/local/instantclient/libocci.so
RUN ln -s /usr/local/instantclient_12_2/sqlplus /usr/bin/sqlplus

RUN echo 'export LD_LIBRARY_PATH="/usr/local/instantclient"' >> /root/.bashrc
RUN echo 'umask 002' >> /root/.bashrc

#RUN echo 'export LD_LIBRARY_PATH="/usr/local/instantclient"'

RUN echo "LD_LIBRARY_PATH=\"/usr/local/instantclient\"" >> /etc/environment \
    && echo "ORACLE_HOME=\"/usr/local/instantclient\"" >> /etc/environment

RUN ldconfig
ENV LD_LIBRARY_PATH /usr/local/instantclient

RUN pecl channel-update pecl.php.net

RUN echo 'instantclient,/usr/local/instantclient' | pecl install oci8-2.2.0
RUN echo "extension=oci8.so" > /usr/local/etc/php/conf.d/php-oci8.ini

RUN apt-get install nano -y

RUN echo "export LD_LIBRARY_PATH=/usr/local/instantclient" >> /etc/apache2/envvars
RUN echo "export ORACLE_HOME=/usr/local/instantclient" >> /etc/apache2/envvars
RUN echo "LD_LIBRARY_PATH=/usr/local/instantclient:\$LD_LIBRARY_PATH" >> /etc/environment

RUN echo "<?php echo phpinfo(); ?>" > /var/www/html/phpinfo.php
RUN echo "<?php echo 'Client Version: ' . oci_client_version(); ?>" > /var/www/html/ocitest.php

RUN echo "service apache2 restart"

RUN php -v

RUN ldconfig -v

RUN ldd /usr/local/lib/php/extensions/no-debug-non-zts-20170718/oci8.so

RUN echo "curl http://localhost/phpinfo.php"
RUN echo "curl http://localhost/ocitest.php"

EXPOSE 80
EXPOSE 443