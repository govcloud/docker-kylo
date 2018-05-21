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
    mkdir -p /opt/nifi/kylo && \
    cp -a install/install-tar/target/kylo/setup/nifi/. /opt/nifi/kylo

##########

FROM govcloud/docker-nifi:1.6.0
LABEL maintainer="William Hearn <william.hearn@canada.ca>"
LABEL site="https://nifi.apache.org/"

ARG UID=1000
ARG GID=1000

ENV JAVA_OPTS="-Xms512M -Xmx1536M"

COPY --from=kylok8sBuild /opt/nifi/kylo /opt/nifi/kylo

USER root

RUN mkdir -p /opt/nifi/activemq && \
    mkdir -p /opt/nifi/drivers && \
    mkdir -p /opt/nifi/lib/app && \
    mkdir -p /opt/nifi/dump && \
    mkdir -p /opt/nifi/drivers

# Kylo JARS
RUN ln -f -s /opt/nifi/kylo/kylo-nifi-core-service-nar-*.nar /opt/nifi/lib/kylo-nifi-core-service-nar.nar && \
    ln -f -s /opt/nifi/kylo/kylo-nifi-standard-services-nar-*.nar /opt/nifi/lib/kylo-nifi-standard-services-nar.nar && \
    ln -f -s /opt/nifi/kylo/kylo-nifi-core-v1-nar-*.nar /opt/nifi/lib/kylo-nifi-core-nar.nar && \
    ln -f -s /opt/nifi/kylo/kylo-nifi-spark-v1-nar-*.nar /opt/nifi/lib/kylo-nifi-spark-nar.nar && \
    ln -f -s /opt/nifi/kylo/kylo-nifi-spark-service-v1-nar-*.nar /opt/nifi/lib/kylo-nifi-spark-service-nar.nar && \
    ln -f -s /opt/nifi/kylo/kylo-nifi-hadoop-v1-nar-*.nar /opt/nifi/lib/kylo-nifi-hadoop-nar.nar && \
    ln -f -s /opt/nifi/kylo/kylo-nifi-hadoop-service-v1-nar-*.nar /opt/nifi/lib/kylo-nifi-hadoop-service-nar.nar && \
    ln -f -s /opt/nifi/kylo/kylo-nifi-elasticsearch-v1-nar-*.nar /opt/nifi/lib/kylo-nifi-elasticsearch-nar.nar && \
    ln -f -s /opt/nifi/kylo/kylo-nifi-provenance-repo-v1.2-nar-*.nar /opt/nifi/lib/kylo-nifi-provenance-repo-nar.nar && \
    ln -f -s /opt/nifi/kylo/kylo-spark-validate-cleanse-v2-*-jar-with-dependencies.jar /opt/nifi/lib/app/kylo-spark-validate-cleanse-jar-with-dependencies.jar && \
    ln -f -s /opt/nifi/kylo/kylo-spark-job-profiler-v2-*-jar-with-dependencies.jar /opt/nifi/lib/app/kylo-spark-job-profiler-jar-with-dependencies.jar && \
    ln -f -s /opt/nifi/kylo/kylo-spark-interpreter-v2-*-jar-with-dependencies.jar /opt/nifi/lib/app/kylo-spark-interpreter-jar-with-dependencies.jar && \
    wget http://central.maven.org/maven2/org/apache/activemq/activemq-all/5.15.0/activemq-all-5.15.0.jar -P /opt/nifi/activemq/ && \
    wget http://central.maven.org/maven2/org/mariadb/jdbc/mariadb-java-client/2.2.4/mariadb-java-client-2.2.4.jar -P /opt/nifi/drivers/

# Impala JDBC driver
RUN wget -q https://downloads.cloudera.com/connectors/impala_jdbc_2.5.42.1062.zip && \
    unzip impala_jdbc_2.5.42.1062.zip && \
    unzip ClouderaImpalaJDBC-2.5.42.1062/ClouderaImpalaJDBC4_2.5.42.zip && \
    mkdir -p /opt/nifi/drivers/impala && \
    mv ClouderaImpalaJDBC4_2.5.42/* /opt/nifi/drivers/impala

# Web HTTP(s) & Socket Site-to-Site Ports
EXPOSE 8080 8443 10000

# Apply configuration and start NiFi
ENTRYPOINT      ["/docker-entrypoint.sh"]
CMD             ["bin/nifi.sh", "run"]
