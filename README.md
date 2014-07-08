# ![](https://gravatar.com/avatar/11d3bc4c3163e3d238d558d5c9d98efe?s=64) aptible/mysql
[![Docker Repository on Quay.io](https://quay.io/repository/aptible/mysql/status)](https://quay.io/repository/aptible/mysql)

MySQL.

## Installation and Usage

    docker pull quay.io/aptible/mysql
    docker run quay.io/aptible/mysql

This will launch a MySQL server that enforces SSL for any TCP connection. **Important note:** Because the key and certificate used for SSL negotiation are included in the Docker image, and shared by all Docker clients running the same version of the image, a MySQL server launched with just `docker run` is **NOT** suitable for production.

To generate a unique key/certificate pair, you have two options:

1. Build directly from the Dockerfile, disabling caching:

        docker build --no-cache .

2. Initialize a new key and certificate in the host volume and mount that directory into the Docker container, as follows:

        cd <host-mountpoint>/ssl
        openssl req -new -newkey rsa:1024 -days 365000 -nodes -x509 \
          -keyout server.key -subj "/CN=MySQL" -out server.crt
        chmod og-rwx server.key
        docker run -v <host-mountpoint>/ssl:/etc/mysql/9.3/ssl -u root \
          quay.io/aptible/mysql chown -R postgres:postgres /etc/mysql/9.3
        docker run -v <host-mountpoint>/ssl:/etc/mysql/9.3/ssl \
          quay.io/aptible/mysql

## Advanced Usage

### Initializing an attached data volume

    id=$(docker run -P -d quay.io/aptible/mysql)
    docker cp $id:/var/lib/mysql <host-mountpoint>
    docker stop $id

### Creating a database user with password

    docker run -v <host-mountpoint>/mysql:/var/lib/mysql quay.io/aptible/mysql sh -c "/etc/init.d/mysql start && psql --command \"CREATE USER aptible WITH SUPERUSER PASSWORD 'password';\""

### Creating a database

    docker run -v <host-mountpoint>/mysql:/var/lib/mysql quay.io/aptible/mysql sh -c "/etc/init.d/mysql start && psql --command \"CREATE DATABASE db;\""

## Available Tags

* `latest`: Currently MySQL 9.3.4

## Tests

Tests are run as part of the `Dockerfile` build. To execute them separately within a container, run:

    bats test

## Deployment

To push the Docker image to Quay, run the following command:

    make release

## Copyright and License

MIT License, see [LICENSE](LICENSE.md) for details.

Copyright (c) 2014 [Aptible](https://www.aptible.com), [Frank Macreery](https://github.com/fancyremarker), and contributors.
