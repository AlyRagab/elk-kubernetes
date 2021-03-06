FROM alpine:3.5

ENV JAVA_HOME=/usr/lib/jvm/default-jvm/jre

RUN apk upgrade --update-cache; \
    apk add openjdk8-jre bash; \
    rm -rf /tmp/* /var/cache/apk/*

############################################################# ELK Docker

EXPOSE 9200 9300

ENV VERSION 5.2.2
ENV GOSU_VERSION 1.7

# Install Elasticsearch.
RUN apk add --update curl ca-certificates sudo jq gnupg && \
  curl -Lso /usr/local/bin/gosu "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-amd64" && \
  curl -Lso /usr/local/bin/gosu.asc "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-amd64.asc" && \
  export GNUPGHOME="$(mktemp -d)" && \
  gpg --keyserver ha.pool.sks-keyservers.net --recv-keys B42F6819007F00F88E364FD4036A9C25BF357DD4 && \
  gpg --batch --verify /usr/local/bin/gosu.asc /usr/local/bin/gosu && \
  rm -r "$GNUPGHOME" /usr/local/bin/gosu.asc && \
  chmod +x /usr/local/bin/gosu && \
  ( curl -Lsj https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-$VERSION.tar.gz | \
  gunzip -c - | tar xf - ) && \
  mv /elasticsearch-$VERSION /elasticsearch && \
  rm -rf $(find /elasticsearch | egrep "(\.(exe|bat)$|sigar/.*(dll|winnt|x86-linux|solaris|ia64|freebsd|macosx))") && \
  apk del gnupg

# Volume for Elasticsearch data
VOLUME ["/data"]

# Copy configuration
COPY config /elasticsearch/config

# Set environment variables defaults
ENV ES_JAVA_OPTS "-Xms512m -Xmx512m"
ENV CLUSTER_NAME elasticsearch-default
ENV NODE_MASTER true
ENV NODE_DATA true
ENV HTTP_ENABLE true
ENV NETWORK_HOST _site_
ENV HTTP_CORS_ENABLE true
ENV HTTP_CORS_ALLOW_ORIGIN *
ENV NUMBER_OF_MASTERS 1
ENV NAMESPACE default
ENV DISCOVERY_SERVICE elasticsearch-discovery

# Install Elasticsearch plug-ins
RUN /elasticsearch/bin/elasticsearch-plugin install x-pack --verbose && /elasticsearch/bin/elasticsearch-plugin install io.fabric8:elasticsearch-cloud-kubernetes:${VERSION} --verbose && /elasticsearch/bin/elasticsearch-plugin install https://github.com/vvanholl/elasticsearch-prometheus-exporter/releases/download/${VERSION}.0/elasticsearch-prometheus-exporter-${VERSION}.0.zip --verbose

# Override elasticsearch.yml config, otherwise plug-in install will fail
ADD config/elasticsearch.yml /elasticsearch/config/elasticsearch.yml

# Copy run script
COPY pre-stop-hook.sh run.sh /

CMD ["/run.sh"]
