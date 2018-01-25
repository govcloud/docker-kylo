FROM govcloud/docker-nifi:latest

MAINTAINER William Hearn <william.hearn@canada.ca>>

ENV GLIBC_VERSION=2.25-r0

ENV JAVA_OPTS="-Xms512M -Xmx1536M"

ENV MAVEN_OPTS="-Xms512M -Xmx1536M"

ENV MAVEN_SKIP_RC="true"

RUN wget -q -O /etc/apk/keys/sgerrand.rsa.pub https://raw.githubusercontent.com/sgerrand/alpine-pkg-glibc/master/sgerrand.rsa.pub && \
    wget -q https://github.com/sgerrand/alpine-pkg-glibc/releases/download/$GLIBC_VERSION/glibc-$GLIBC_VERSION.apk && \
    apk add --update --no-cache glibc-$GLIBC_VERSION.apk \
                                git \
                                maven && \
    echo @community http://dl-cdn.alpinelinux.org/alpine/edge/community | tee -a /etc/apk/repositories && \
    apk --update add rpm && \
    rm glibc-$GLIBC_VERSION.apk

RUN git clone https://github.com/fabiannecci/kylo.git /opt/kylok8s && \
    cd /opt/kylok8s && \
    git checkout teamdigitale && \
    mvn clean install -DskipTests -DskipDebian && \
    mkdir install/install-tar/target/kylo && \
    tar -C install/install-tar/target/kylo -xvf install/install-tar/target/kylo-*-dependencies.tar.gz && \
    cp -a install/install-tar/target/kylo/setup/nifi/. /opt/nifi/kylo && \
    mkdir -p /opt/nifi/activemq && \
    mkdir -p /opt/nifi/drivers && \
    mkdir -p /opt/nifi/lib/app && \
    mkdir -p /opt/nifi/dump && \
	mvn clean

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
    ln -f -s /opt/nifi/kylo/kylo-spark-validate-cleanse-spark-v2-0.8.4-jar-with-dependencies.jar /opt/nifi/lib/app/kylo-spark-validate-cleanse-jar-with-dependencies.jar && \
    ln -f -s /opt/nifi/kylo/kylo-spark-job-profiler-spark-v2-*-jar-with-dependencies.jar /opt/nifi/lib/app/kylo-spark-job-profiler-jar-with-dependencies.jar && \
    ln -f -s /opt/nifi/kylo/kylo-spark-interpreter-spark-v2-*-jar-with-dependencies.jar /opt/nifi/lib/app/kylo-spark-interpreter-jar-with-dependencies.jar && \
    wget http://central.maven.org/maven2/org/apache/activemq/activemq-all/5.15.0/activemq-all-5.15.0.jar -P /opt/nifi/activemq/ && \
    wget http://central.maven.org/maven2/org/mariadb/jdbc/mariadb-java-client/2.1.0/mariadb-java-client-2.1.0.jar -P /opt/nifi/drivers/

RUN apk --update add krb5 \
                     krb5-libs \
                     coreutils \
                     cyrus-sasl \
                     python \
                     python3 \
                     python3-dev \
                     sudo && \
    python3 -m ensurepip && \
    rm -r /usr/lib/python*/ensurepip && \
    pip3 install --upgrade pip setuptools
