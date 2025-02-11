# Installing Xdebug

The default Docker stack is shipped without a Xdebug stage.
It's easy though to add [Xdebug](https://xdebug.org/) to your project, for development purposes such as debugging tests or API requests remotely.

## Add a Debug Stage to the Dockerfile

To avoid deploying Symfony Docker to production with an active Xdebug extension,
it's recommended to add a custom stage to the end of the `Dockerfile`.

```Dockerfile
# Dockerfile
FROM symfony_php as symfony_php_debug

ARG XDEBUG_VERSION=3.0.4
RUN set -eux; \
	apk add --no-cache --virtual .build-deps $PHPIZE_DEPS; \
	pecl install xdebug-$XDEBUG_VERSION; \
	docker-php-ext-enable xdebug; \
	apk del .build-deps
```

## Configure Xdebug with Docker Compose Override

Using an [override](https://docs.docker.com/compose/reference/overview/#specifying-multiple-compose-files) file named `docker-compose.debug.yaml` ensures that the production
configuration remains untouched.

As example, an override could look like this:

```yaml
# docker-compose.debug.yaml
version: "3.4"

services:
  php:
    build:
      context: .
      target: symfony_php_debug
    environment:
      # See https://docs.docker.com/docker-for-mac/networking/#i-want-to-connect-from-a-container-to-a-service-on-the-host
      # See https://github.com/docker/for-linux/issues/264
      # The `client_host` below may optionally be replaced with `discover_client_host=yes`
      # Add `start_with_request=yes` to start debug session on each request
      XDEBUG_CONFIG: >-
        client_host=host.docker.internal
      XDEBUG_MODE: debug
      # This should correspond to the server declared in PHPStorm `Preferences | Languages & Frameworks | PHP | Servers`
      # Then PHPStorm will use the corresponding path mappings
      PHP_IDE_CONFIG: serverName=symfony
```

Then run:

```console
docker-compose -f docker-compose.yml -f docker-compose.debug.yml up -d
```

## Troubleshooting

Inspect the installation with the following command. The requested Xdebug version should be displayed in the output.

```console
$ docker-compose exec php php --version

PHP ...
    with Xdebug v3.0.4 ...
```
