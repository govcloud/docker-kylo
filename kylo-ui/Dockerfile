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

FROM openjdk:8-jdk AS kylok8sBuild
LABEL maintainer="Apache NiFi <dev@nifi.apache.org>"
LABEL site="https://nifi.apache.org"

ARG UID=1000
ARG GID=1000

ENV JAVA_OPTS="-Xms512M -Xmx1536M"
ENV MAVEN_OPTS="-Xms512M -Xmx1536M"
ENV MAVEN_SKIP_RC="true"

USER root

RUN apt-get update && \
    apt-get install -y maven \
                       rpm

RUN git clone https://github.com/teradata/kylo.git /opt/kylok8s && \
    cd /opt/kylok8s && \
    git checkout tags/v0.9.1 && \
    mvn clean install -DskipTests

RUN cd /opt/kylok8s && \
    mkdir install/install-tar/target/kylo && \
    tar -C install/install-tar/target/kylo -xvf install/install-tar/target/kylo-*-dependencies.tar.gz && \
    mkdir -p /opt/kylo/kylo-ui && \
    cp -a install/install-tar/target/kylo/kylo-ui/lib /opt/kylo/kylo-ui/ && \
    cp -a install/install-tar/target/kylo/kylo-ui/plugin /opt/kylo/kylo-ui/ && \
    cp -a install/install-tar/target/kylo/bin /opt/kylo && \
    cp -a install/install-tar/target/kylo/lib /opt/kylo

##########

FROM openjdk:8-jre
LABEL maintainer="William Hearn <william.hearn@canada.ca>"
LABEL site="https://nifi.apache.org/"

ARG UID=1000
ARG GID=1000

ENV JAVA_OPTS="-Xms512M -Xmx1536M"

COPY --from=kylok8sBuild /opt/kylo /opt/kylo

EXPOSE 8400

CMD ["java", "-cp", "/opt/kylo/kylo-ui/conf:/opt/kylo/kylo-ui/lib/*:/opt/kylo/kylo-ui/plugin/*", "com.thinkbiganalytics.KyloUiApplication"]
