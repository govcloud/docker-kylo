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
                                mysql-client \
                                ca-certificates \
                                bash \
                                java-snappy \
                                python \
                                py-pip \
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
    mkdir -p /opt/kylo/kylo-ui && \
    cp -a install/install-tar/target/kylo/kylo-ui/lib /opt/kylo/kylo-ui/ && \
    cp -a install/install-tar/target/kylo/kylo-ui/plugin /opt/kylo/kylo-ui/ && \
    cp -a install/install-tar/target/kylo/bin /opt/kylo && \
    cp -a install/install-tar/target/kylo/lib /opt/kylo && \
    mvn clean

EXPOSE 8400

CMD ["java", "-cp", "/opt/kylo/kylo-ui/conf:/opt/kylo/kylo-ui/lib/*:/opt/kylo/kylo-ui/plugin/*", "com.thinkbiganalytics.KyloUiApplication"]
