FROM openjdk:8-alpine

MAINTAINER William Hearn <william.hearn@canada.ca>>

ENV GLIBC_VERSION=2.25-r0

ENV JAVA_OPTS="-Xms512M -Xmx1536M"

ENV MAVEN_OPTS="-Xms512M -Xmx1536M"

ENV MAVEN_SKIP_RC="true"

ENV SPARK_HOME=/opt/cloudera/parcels/SPARK2/lib/spark2

RUN wget -q -O /etc/apk/keys/sgerrand.rsa.pub https://raw.githubusercontent.com/sgerrand/alpine-pkg-glibc/master/sgerrand.rsa.pub && \
    wget -q https://github.com/sgerrand/alpine-pkg-glibc/releases/download/$GLIBC_VERSION/glibc-$GLIBC_VERSION.apk && \
    apk add --update --no-cache wget \
                                curl \
                                mysql-client \
                                ca-certificates \
                                bash \
                                java-snappy \
                                python \
                                py-pip \
                                krb5 \
                                sudo \
                                coreutils \
                                git \
                                maven \
                                rpm && \
    update-ca-certificates && \
    echo @community http://dl-cdn.alpinelinux.org/alpine/edge/community | tee -a /etc/apk/repositories && \
    rm glibc-$GLIBC_VERSION.apk

RUN git clone https://github.com/fabiannecci/kylo.git /opt/kylok8s && \
    cd /opt/kylok8s && \
    git checkout teamdigitale && \
    mvn clean install -DskipTests -DskipDebian && \
    mkdir install/install-tar/target/kylo && \
    tar -C install/install-tar/target/kylo -xvf install/install-tar/target/kylo-*-dependencies.tar.gz && \
    mkdir -p /opt/kylo/kylo-services && \
    if [ ! -f install/install-tar/target/kylo/kylo-services/lib/postgresql-42.1.4.jar ] ;then curl -o install/install-tar/target/kylo/kylo-services/lib/postgresql-42.1.4.jar http://central.maven.org/maven2/org/postgresql/postgresql/42.1.4/postgresql-42.1.4.jar ;fi  && \
    cp -a install/install-tar/target/kylo/kylo-services/lib /opt/kylo/kylo-services/ && \
    cp -a install/install-tar/target/kylo/kylo-services/plugin /opt/kylo/kylo-services/ && \
    cp -a install/install-tar/target/kylo/bin /opt/kylo && \
    cp -a install/install-tar/target/kylo/lib /opt/kylo && \
    mvn clean

RUN rm -f /opt/kylo/kylo-services/lib/jetty* && \
    rm -f /opt/kylo/kylo-services/lib/servlet-api* && \
    adduser -S -s /bin/bash kylo && \
    addgroup kylo && \
    adduser kylo kylo && \
    chown -R kylo:kylo /opt/kylo

EXPOSE 8420

CMD ["java", "-cp", "/opt/kylo/kylo-services/conf:/opt/kylo/kylo-services/lib/*:/opt/kylo/kylo-services/lib/nifi-v1.2/*:/opt/kylo/kylo-services/plugin/*", "com.thinkbiganalytics.server.KyloServerApplication"]
