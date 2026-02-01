# PHP 7.4 with Apache and Oracle Instant Client 12.2.0.1.0

This image is built from [official PHP images](https://hub.docker.com/_/php/). It includes Oracle Instant Client 12.2.0.1.0 for connecting to Oracle databases.

The docker image can be built locally using the Dockerfile in the root directory.

## Building the Image

```bash
docker build -t php7-oci8 .
```

## Running the Container

```bash
docker-compose up
```

Or run directly:

```bash
docker run -d -p 7000:80 -p 7443:443 -v $(pwd)/www:/var/www/html php7-oci8
```

## Installing OCI8 Extension

The base image includes Oracle Instant Client 12.2.0.1.0, but the OCI8 PHP extension needs to be installed separately. 

To install OCI8 extension (requires access to pecl.php.net):

```bash
docker exec -it <container_id> bash
pecl install oci8-2.2.0
echo "extension=oci8.so" > /usr/local/etc/php/conf.d/php-oci8.ini
apache2ctl restart
```

## Notes

- PHP Version: 7.4.33
- Oracle Instant Client: 12.2.0.1.0
- Compatible OCI8 version: 2.2.0
- The image includes necessary libraries and symlinks for Oracle connectivity
- Apache with SSL enabled (self-signed certificate)
- LDAP support enabled

## Original Source

Original repository references:
- [hub.docker.com/r/adrianharabula/php7-with-oci8](https://hub.docker.com/r/adrianharabula/php7-with-oci8)
- [docker-apache-oracle-php](https://github.com/adrianharabula/daopstack)
