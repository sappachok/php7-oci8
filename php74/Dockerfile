FROM php:7.4-apache

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
        libzip-dev 
    #&& docker-php-ext-install zip \
    #&& docker-php-ext-install zip \
    #&& docker-php-ext-install -j$(nproc) iconv gettext \
    #&& docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    #&& docker-php-ext-install -j$(nproc) gd

# Install zip
RUN apt-get update && \
	apt-get install -y \
        libzip-dev
        #&& docker-php-ext-install zip \
	#&& docker-php-ext-install -j$(nproc) iconv gettext \
	#&& docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
	#&& docker-php-ext-install -j$(nproc) gd


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

# Oracle instantclient
ADD ./instantclient/21.1.0.0.0/instantclient-basic-linux.x64-21.1.0.0.0.zip /tmp/
ADD ./instantclient/21.1.0.0.0/instantclient-sdk-linux.x64-21.1.0.0.0.zip /tmp/
ADD ./instantclient/21.1.0.0.0/instantclient-sqlplus-linux.x64-21.1.0.0.0.zip /tmp/

RUN mkdir /usr/lib/oracle
RUN mkdir /usr/lib/oracle/21.1
RUN mkdir /usr/lib/oracle/21.1/client64

RUN cp /tmp/instantclient-basic-linux.x64-21.1.0.0.0.zip /usr/lib/oracle/21.1/client64
RUN cp /tmp/instantclient-sdk-linux.x64-21.1.0.0.0.zip /usr/lib/oracle/21.1/client64

RUN cd /usr/lib/oracle/21.1/client64
RUN ls

WORKDIR /usr/lib/oracle/21.1/client64

RUN unzip instantclient-basic-linux.x64-21.1.0.0.0.zip
RUN unzip instantclient-sdk-linux.x64-21.1.0.0.0.zip

RUN mv instantclient_21_1 lib
RUN cd /usr/lib/oracle/21.1/client64/lib/
RUN ln -s libclntsh.so.21.1 libclntsh.so
RUN ln -s libocci.so.21.1 libocci.so

RUN echo /usr/lib/oracle/21.1/client64/lib > /etc/ld.so.conf.d/oracle.conf

RUN ldconfig

RUN pecl channel-update pecl.php.net

RUN echo 'instantclient,/usr/local/instantclient' | pecl install oci8
# echo 'instantclient,/usr/local/instantclient' | pecl install oci8-2.2.0

RUN echo "curl http://localhost/phpinfo.php"
RUN echo "curl http://localhost/ocitest.php"

EXPOSE 80
EXPOSE 443