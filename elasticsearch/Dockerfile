################################################################################
# This Dockerfile was generated from the template at distribution/src/docker/Dockerfile
#
# Beginning of multi stage Dockerfile
################################################################################

################################################################################
# Build stage 0 `builder`:
# Extract elasticsearch artifact
# Install required plugins
# Set gid=0 and make group perms==owner perms
################################################################################

FROM centos:7 AS builder

RUN for iter in {1..10}; do yum update --setopt=tsflags=nodocs -y &&     yum install --setopt=tsflags=nodocs -y gzip shadow-utils tar &&     yum clean all && exit_code=0 && break || exit_code=$? && echo "yum error: retry $iter in 10s" && sleep 10; done;     (exit $exit_code)

ENV PATH /usr/share/elasticsearch/bin:$PATH

RUN groupadd -g 1000 elasticsearch &&     adduser -u 1000 -g 1000 -d /usr/share/elasticsearch elasticsearch

WORKDIR /usr/share/elasticsearch

RUN curl -sL https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-oss-7.6.2-no-jdk-linux-x86_64.tar.gz | tar --strip-components=1 -zxf -
RUN grep ES_DISTRIBUTION_TYPE=tar /usr/share/elasticsearch/bin/elasticsearch-env     && sed -i -e 's/ES_DISTRIBUTION_TYPE=tar/ES_DISTRIBUTION_TYPE=docker/' /usr/share/elasticsearch/bin/elasticsearch-env
RUN curl -sL http://download.bell-sw.com/java/13.0.2+9/bellsoft-jdk13.0.2+9-linux-aarch64.tar.gz | tar -C /opt -zxf -
RUN mkdir -p config data logs
RUN chmod 0775 config data logs
COPY config/elasticsearch.yml config/log4j2.properties config/
RUN chmod 0660 config/elasticsearch.yml config/log4j2.properties

################################################################################
# Build stage 1 (the actual elasticsearch image):
# Copy elasticsearch from stage 0
# Add entrypoint
################################################################################

FROM centos:7

ENV ELASTIC_CONTAINER true
ENV JAVA_HOME /opt/jdk-13.0.2

RUN for iter in {1..10}; do yum update --setopt=tsflags=nodocs -y &&     yum install --setopt=tsflags=nodocs -y nc shadow-utils zip unzip &&     yum clean all && exit_code=0 && break || exit_code=$? && echo "yum error: retry $iter in 10s" && sleep 10; done;     (exit $exit_code)

RUN groupadd -g 1000 elasticsearch &&     adduser -u 1000 -g 1000 -G 0 -d /usr/share/elasticsearch elasticsearch &&     chmod 0775 /usr/share/elasticsearch &&     chgrp 0 /usr/share/elasticsearch

WORKDIR /usr/share/elasticsearch
COPY --from=builder --chown=1000:0 /usr/share/elasticsearch /usr/share/elasticsearch
COPY --from=builder --chown=1000:0 /opt/jdk-13.0.2 /opt/jdk-13.0.2

# Replace OpenJDK's built-in CA certificate keystore with the one from the OS
# vendor. The latter is superior in several ways.
# REF: https://github.com/elastic/elasticsearch-docker/issues/171
RUN ln -sf /etc/pki/ca-trust/extracted/java/cacerts /opt/jdk-13.0.2/lib/security/cacerts

ENV PATH /usr/share/elasticsearch/bin:$PATH

COPY --chown=1000:0 bin/docker-entrypoint.sh /usr/local/bin/docker-entrypoint.sh

# Openshift overrides USER and uses ones with randomly uid>1024 and gid=0
# Allow ENTRYPOINT (and ES) to run even with a different user
RUN chgrp 0 /usr/local/bin/docker-entrypoint.sh &&     chmod g=u /etc/passwd &&     chmod 0775 /usr/local/bin/docker-entrypoint.sh

EXPOSE 9200 9300

LABEL org.label-schema.build-date="2020-04-10T20:50:35.705861Z"   org.label-schema.license="Apache-2.0"   org.label-schema.name="Elasticsearch"   org.label-schema.schema-version="1.0"   org.label-schema.url="https://www.elastic.co/products/elasticsearch"   org.label-schema.usage="https://www.elastic.co/guide/en/elasticsearch/reference/index.html"   org.label-schema.vcs-ref="ef48eb35cf30adf4db14086e8aabd07ef6fb113f"   org.label-schema.vcs-url="https://github.com/elastic/elasticsearch"   org.label-schema.vendor="Elastic"   org.label-schema.version="7.6.2-SNAPSHOT"   org.opencontainers.image.created="2020-04-10T20:50:35.705861Z"   org.opencontainers.image.documentation="https://www.elastic.co/guide/en/elasticsearch/reference/index.html"   org.opencontainers.image.licenses="Apache-2.0"   org.opencontainers.image.revision="ef48eb35cf30adf4db14086e8aabd07ef6fb113f"   org.opencontainers.image.source="https://github.com/elastic/elasticsearch"   org.opencontainers.image.title="Elasticsearch"   org.opencontainers.image.url="https://www.elastic.co/products/elasticsearch"   org.opencontainers.image.vendor="Elastic"   org.opencontainers.image.version="7.6.2-SNAPSHOT"

ENTRYPOINT ["/usr/local/bin/docker-entrypoint.sh"]
# Dummy overridable parameter parsed by entrypoint
CMD ["eswrapper"]

################################################################################
# End of multi-stage Dockerfile
################################################################################
