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

RUN git clone https://github.com/italia/kylo.git /opt/kylok8s && \
    cd /opt/kylok8s && \
    git checkout release/0.8.4-daf-kylo && \
    git pull && \
    mvn clean install -DskipTests

RUN cd /opt/kylok8s && \
    mkdir install/install-tar/target/kylo && \
    tar -C install/install-tar/target/kylo -xvf install/install-tar/target/kylo-*-dependencies.tar.gz && \
    mkdir -p /opt/kylo/kylo-services && \
    if [ ! -f install/install-tar/target/kylo/kylo-services/lib/postgresql-42.1.4.jar ] ;then curl -o install/install-tar/target/kylo/kylo-services/lib/postgresql-42.1.4.jar http://central.maven.org/maven2/org/postgresql/postgresql/42.1.4/postgresql-42.1.4.jar ;fi  && \
    cp -a install/install-tar/target/kylo/kylo-services/lib /opt/kylo/kylo-services/ && \
    cp -a install/install-tar/target/kylo/kylo-services/plugin /opt/kylo/kylo-services/ && \
    cp -a install/install-tar/target/kylo/bin /opt/kylo && \
    cp -a install/install-tar/target/kylo/lib /opt/kylo

##########

FROM openjdk:8-jre
LABEL maintainer="William Hearn <william.hearn@canada.ca>"
LABEL site="https://nifi.apache.org/"

ARG UID=1000
ARG GID=1000

ENV JAVA_OPTS="-Xms512M -Xmx1536M"

ENV SPARK_HOME=/opt/cloudera/parcels/SPARK2/lib/spark2

COPY --from=kylok8sBuild /opt/kylo /opt/kylo

# USER root

# RUN rm -f /opt/kylo/kylo-services/lib/jetty* && \
#     rm -f /opt/kylo/kylo-services/lib/servlet-api* && \
#     adduser --system --shell /bin/bash kylo && \
#     addgroup kylo && \
#     adduser kylo kylo && \
#     chown -R kylo:kylo /opt/kylo

# Impala JDBC driver
# RUN wget -q https://downloads.cloudera.com/connectors/impala_jdbc_2.5.42.1062.zip && \
#     unzip impala_jdbc_2.5.42.1062.zip && \
#     unzip ClouderaImpalaJDBC-2.5.42.1062/ClouderaImpalaJDBC4_2.5.42.zip && \
#     mv ClouderaImpalaJDBC4_2.5.42/ImpalaJDBC4.jar /opt/kylo/kylo-services/plugin && \
#     mv ClouderaImpalaJDBC4_2.5.42/TCLIServiceClient.jar /opt/kylo/kylo-services/plugin && \
#     mv ClouderaImpalaJDBC4_2.5.42/ql.jar /opt/kylo/kylo-services/plugin

EXPOSE 8420

CMD ["java", "-cp", "/opt/kylo/kylo-services/conf:/opt/kylo/kylo-services/lib/*:/opt/kylo/kylo-services/lib/nifi-v1.2/*:/opt/kylo/kylo-services/plugin/*", "com.thinkbiganalytics.server.KyloServerApplication"]
