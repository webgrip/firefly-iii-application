ARG DATAIMPORTER_VERSION=1.7.10
FROM fireflyiii/data-importer:version-${DATAIMPORTER_VERSION}

USER root
RUN apt-get update \
  && apt-get upgrade -y \
  && apt-get autoremove -y --purge \
  && rm -rf /var/lib/apt/lists/*
USER www-data
