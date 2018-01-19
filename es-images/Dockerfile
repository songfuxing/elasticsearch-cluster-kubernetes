#FROM quay.io/pires/docker-elasticsearch:6.1.1
FROM elasticsearch:5.6.4-alpine

#MAINTAINER pjpires@gmail.com

# Override config, otherwise plug-in install will fail
ADD config /usr/share/elasticsearch/config

# Set environment
ENV DISCOVERY_SERVICE elasticsearch-discovery

# Kubernetes requires swap is turned off, so memory lock is redundant
ENV MEMORY_LOCK false
#COPY jieba-es-plugin.tar.gz /elasticsearch/plugins
COPY ./jieba-es-plugin.tar.gz /usr/share/elasticsearch/plugins

WORKDIR /usr/share/elasticsearch/plugins
RUN tar xzf jieba-es-plugin.tar.gz && \
    rm jieba-es-plugin.tar.gz && \
    sed -i -e "s/5\.6\.1/5.6.4/g" jieba/plugin-descriptor.properties

WORKDIR /usr/share/elasticsearch

#WORKDIR /elasticsearch/plugins
#RUN tar xzf jieba-es-plugin.tar.gz && \
#    rm jieba-es-plugin.tar.gz && \
#    sed -i -e "s/5\.6\.1/5.6.4/g" jieba/plugin-descriptor.properties
