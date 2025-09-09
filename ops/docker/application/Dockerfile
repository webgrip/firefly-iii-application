ARG FIREFLY_VERSION=6.3.2
FROM fireflyiii/core:version-${FIREFLY_VERSION}

SHELL ["/bin/sh", "-euxo", "pipefail"]

COPY ./ops/docker/application/php.ini /usr/local/etc/php/conf.d/zz-custom.ini

HEALTHCHECK --interval=60s --timeout=10s --start-period=40s --retries=3 \
  CMD php artisan firefly-iii:output-version >/dev/null 2>&1 || exit 1

USER www-data
