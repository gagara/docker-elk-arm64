################################################################################
# This Dockerfile was generated from the template at .tedi/Dockerfile.j2
#
# Beginning of multi stage Dockerfile
################################################################################

################################################################################
# Build stage 0 `prep_files`:
# Extract elasticsearch artifact
# Install required plugins
# Set gid=0 and make group perms==owner perms
################################################################################

FROM centos:7 AS prep_files

ENV PATH /usr/share/elasticsearch/bin:$PATH
ENV JAVA_HOME /opt/jdk-11.0.1

RUN curl -sL https://github.com/bell-sw/Liberica/releases/download/11.0.1/bellsoft-jdk11.0.1-linux-aarch64.tar.gz | tar -C /opt -zxf -
RUN curl -sL https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-oss-6.6.2.tar.gz -o /opt/elasticsearch-oss-6.6.2.tar.gz
RUN curl -sL https://artifacts.elastic.co/downloads/elasticsearch-plugins/ingest-geoip/ingest-geoip-6.6.2.zip -o /opt/ingest-geoip.zip
RUN curl -sL https://artifacts.elastic.co/downloads/elasticsearch-plugins/ingest-user-agent/ingest-user-agent-6.6.2.zip -o /opt/ingest-user-agent.zip

# Replace OpenJDK's built-in CA certificate keystore with the one from the OS
# vendor. The latter is superior in several ways.
# REF: https://github.com/elastic/elasticsearch-docker/issues/171
RUN ln -sf /etc/pki/ca-trust/extracted/java/cacerts /opt/jdk-11.0.1/lib/security/cacerts

RUN yum install -y unzip which

RUN groupadd -g 1000 elasticsearch && \
    adduser -u 1000 -g 1000 -d /usr/share/elasticsearch elasticsearch

WORKDIR /usr/share/elasticsearch

RUN tar zxf /opt/elasticsearch-oss-6.6.2.tar.gz --strip-components=1
RUN elasticsearch-plugin install --batch file:///opt/ingest-user-agent.zip
RUN elasticsearch-plugin install --batch file:///opt/ingest-geoip.zip
RUN mkdir -p config data logs
RUN chmod 0775 config data logs
COPY elasticsearch.yml log4j2.properties config/

# remove unsupported JVM opts in ARM64
RUN mv config/jvm.options config/jvm.options.orig && \
    cat config/jvm.options.orig | sed -e 's/\(.*UseAVX=.*\)/#\1/' > config/jvm.options

################################################################################
# Build stage 1 (the actual elasticsearch image):
# Copy elasticsearch from stage 0
# Add entrypoint
################################################################################

FROM centos:7

ENV ELASTIC_CONTAINER true
ENV JAVA_HOME /opt/jdk-11.0.1

COPY --from=prep_files /opt/jdk-11.0.1 /opt/jdk-11.0.1

RUN yum update -y && \
    yum install -y nc unzip wget which && \
    yum clean all

RUN groupadd -g 1000 elasticsearch && \
    adduser -u 1000 -g 1000 -G 0 -d /usr/share/elasticsearch elasticsearch && \
    chmod 0775 /usr/share/elasticsearch && \
    chgrp 0 /usr/share/elasticsearch

WORKDIR /usr/share/elasticsearch
COPY --from=prep_files --chown=1000:0 /usr/share/elasticsearch /usr/share/elasticsearch
ENV PATH /usr/share/elasticsearch/bin:$PATH

COPY --chown=1000:0 bin/docker-entrypoint.sh /usr/local/bin/docker-entrypoint.sh

# Openshift overrides USER and uses ones with randomly uid>1024 and gid=0
# Allow ENTRYPOINT (and ES) to run even with a different user
RUN chgrp 0 /usr/local/bin/docker-entrypoint.sh && \
    chmod g=u /etc/passwd && \
    chmod 0775 /usr/local/bin/docker-entrypoint.sh

EXPOSE 9200 9300

LABEL org.label-schema.schema-version="1.0" \
  org.label-schema.vendor="Elastic" \
  org.label-schema.name="elasticsearch" \
  org.label-schema.version="6.6.2" \
  org.label-schema.url="https://www.elastic.co/products/elasticsearch" \
  org.label-schema.vcs-url="https://github.com/elastic/elasticsearch-docker" \
license="Apache-2.0"
ENTRYPOINT ["/usr/local/bin/docker-entrypoint.sh"]
# Dummy overridable parameter parsed by entrypoint
CMD ["eswrapper"]

################################################################################
# End of multi-stage Dockerfile
################################################################################
