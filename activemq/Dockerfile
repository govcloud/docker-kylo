FROM openjdk:alpine

MAINTAINER William Hearn <william.hearn@canada.ca>>

ENV ACTIVEMQ_VERSION 5.15.1

ENV ACTIVEMQ_HOME /opt/activemq

ENV SUPERVISOR_VERSION=3.3.1

RUN apk add --update --no-cache wget \
                                python \
                                py-pip

RUN pip install supervisor==$SUPERVISOR_VERSION && \
    rm -rf /root/.cache

WORKDIR /opt

RUN wget -qO- https://archive.apache.org/dist/activemq/$ACTIVEMQ_VERSION/apache-activemq-$ACTIVEMQ_VERSION-bin.tar.gz | tar -xz && \
    mv apache-activemq-$ACTIVEMQ_VERSION $ACTIVEMQ_HOME

EXPOSE 61616 672 61613 1883 61614 8161

ENTRYPOINT $ACTIVEMQ_HOME/bin/activemq console
