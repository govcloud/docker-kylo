# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements. See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership. The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License. You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied. See the License for the
# specific language governing permissions and limitations
# under the License.
#

FROM openjdk:alpine
LABEL maintainer="William Hearn <william.hearn@canada.ca>"
LABEL site="https://kylo.io"

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
