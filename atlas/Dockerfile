FROM ubuntu:18.04

ENV ATLAS_VERSION 2.1.0
ENV TARBALL apache-atlas-${ATLAS_VERSION}-sources.tar.gz
ENV ATLAS_SOURCE https://dist.apache.org/repos/dist/release/atlas/${ATLAS_VERSION}/${TARBALL}
ENV MAVEN_OPTS="$MAVEN_OPTS -Xms2g -Xmx2g -Dorg.slf4j.simpleLogger.log.org.apache.maven.cli.transfer.Slf4jMavenTransferListener=warn"

ENV ATLAS_HOME="/opt/atlas"
#ENV HADOOP_HOME=""

RUN echo 'debconf debconf/frontend select Noninteractive' | debconf-set-selections

RUN apt-get update \
    && apt-get -y upgrade \
    && apt-get -y install apt-utils \
    && apt-get -y --no-install-recommends install \
        apt-utils \
        maven \
        wget \
        git \
        python \
        openjdk-8-jdk-headless \
        patch \
        unzip \
        curl

RUN cd /tmp/ && \
    wget -qO- ${ATLAS_SOURCE} > apache-atlas-sources-${ATLAS_VERSION}.tar.gz && \
    tar -xvzf apache-atlas-sources-${ATLAS_VERSION}.tar.gz 

# build from source (FIXME)
#RUN cd /tmp/apache-atlas-sources-${ATLAS_VERSION} && \
#    mvn clean -DskipTests package -Pdist,embedded-hbase-solr,berkeley-elasticsearch -B

# OR download pre built
RUN cd /tmp/apache-atlas-sources-${ATLAS_VERSION} && \
    mkdir -p distro/target/ && \
    wget -qO- https://5a-scp-test.s3.amazonaws.com/apache-atlas-2.1.0-ec2.tar.gz > distro/target/apache-atlas-${ATLAS_VERSION}-server.tar.gz

# copy built version for output folder
RUN tar -xzf /tmp/apache-atlas-sources-${ATLAS_VERSION}/distro/target/apache-atlas-${ATLAS_VERSION}-server.tar.gz -C /opt && \
    mv /opt/apache-atlas-${ATLAS_VERSION}/ ${ATLAS_HOME}

VOLUME ["/opt/atlas/conf", "/opt/atlas/logs"]

# input configuration files
COPY atlas_start.py.patch atlas_config.py.patch ${ATLAS_HOME}/bin/

RUN cd ${ATLAS_HOME}/bin \
    && patch -b -f < atlas_start.py.patch \
    && patch -b -f < atlas_config.py.patch

COPY conf/hbase/hbase-site.xml.template ${ATLAS_HOME}/conf/hbase/hbase-site.xml.template
COPY conf/atlas-env.sh ${ATLAS_HOME}/conf/atlas-env.sh
COPY conf/atlas-log4j.xml ${ATLAS_HOME}/conf/atlas-log4j.xml
COPY atlas-application.properties ${ATLAS_HOME}/conf/atlas-application.properties

#RUN cd ${ATLAS_HOME} \
#    && ./bin/atlas_start.py -setup || true

# run one time during build to reduce startup time
#RUN cd ${ATLAS_HOME} \
#    && ./bin/atlas_start.py & \
#    touch ${ATLAS_HOME}/logs/application.log \
#    tail -f ${ATLAS_HOME}/logs/application.log | sed '/AtlasAuthenticationFilter.init(filterConfig=null)/ q' \
#    && sleep 10 \
#    && ${ATLAS_HOME}/bin/atlas_stop.py

# clean apt
#RUN apt-get clean && \
#    rm -rf /var/lib/apt/lists/*

# clean source files
#RUN rm /tmp/apache-atlas-sources-${ATLAS_VERSION}.tar.gz && \
#    rm -rf /tmp/apache-atlas-sources-${ATLAS_VERSION}/

# Install ES jar
#RUN mkdir ${ATLAS_HOME}/libext && \
#    wget -qO- https://repo1.maven.org/maven2/org/janusgraph/janusgraph-es/0.5.1/janusgraph-es-0.5.1.jar > ${ATLAS_HOME}/libext/janusgraph-es-0.5.1.jar && \
#    wget -qO- https://repo1.maven.org/maven2/org/elasticsearch/client/elasticsearch-rest-client/5.6.4/elasticsearch-rest-client-5.6.4.jar > ${ATLAS_HOME}/libext/elasticsearch-rest-client-5.6.4.jar && \
#    wget -qO- https://repo1.maven.org/maven2/org/elasticsearch/client/elasticsearch-rest-high-level-client/5.6.4/elasticsearch-rest-high-level-client-5.6.4.jar > ${ATLAS_HOME}/libext/elasticsearch-rest-high-level-client-5.6.4.jar && \
#    wget -qO- https://repo1.maven.org/maven2/org/elasticsearch/elasticsearch-x-content/7.6.0/elasticsearch-x-content-7.6.0.jar > ${ATLAS_HOME}/libext/elasticsearch-x-content-7.6.0.jar && \
#    wget -qO- https://repo1.maven.org/maven2/com/sleepycat/je/18.3.12/je-18.3.12.jar > ${ATLAS_HOME}/libext/je-18.3.12.jar

# Copy config files
#COPY configuration/atlas-cassandra-es.properties ${ATLAS_HOME}/conf/atlas-application.properties
#COPY configuration/atlas-log4j.xml ${ATLAS_HOME}/conf/

# Atlas URI Port
EXPOSE 21000

WORKDIR ${ATLAS_HOME}

CMD ["/opt/atlas/bin/atlas_start.py"]
