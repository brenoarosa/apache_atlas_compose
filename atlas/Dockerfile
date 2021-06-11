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

# Fixes dependency version
RUN cd /tmp/apache-atlas-sources-${ATLAS_VERSION} \
    && sed -i "s/<zookeeper.version>3.4.6<\/zookeeper.version>/<zookeeper.version>3.5.5<\/zookeeper.version>/" pom.xml \
    && sed -i "s/<elasticsearch.version>5.6.4<\/elasticsearch.version>/<elasticsearch.version>6.6.0<\/elasticsearch.version>/" pom.xml

# build from source (FIXME)
RUN cd /tmp/apache-atlas-sources-${ATLAS_VERSION} && \
    mvn clean -DskipTests package -Pdist -B

# copy built version for output folder
RUN tar -xzf /tmp/apache-atlas-sources-${ATLAS_VERSION}/distro/target/apache-atlas-${ATLAS_VERSION}-server.tar.gz -C /opt && \
    mv /opt/apache-atlas-${ATLAS_VERSION}/ ${ATLAS_HOME}

# input configuration files
COPY bin/atlas_start.py.patch ${ATLAS_HOME}/bin/
COPY bin/atlas_config.py.patch ${ATLAS_HOME}/bin/

RUN cd ${ATLAS_HOME}/bin \
    && patch -b -f < atlas_start.py.patch \
    && patch -b -f < atlas_config.py.patch

VOLUME ["/opt/atlas/conf", "/opt/atlas/logs"]

#COPY conf/hbase/hbase-site.xml.template ${ATLAS_HOME}/conf/hbase/hbase-site.xml.template
COPY conf/atlas-env.sh ${ATLAS_HOME}/conf/atlas-env.sh
COPY conf/atlas-log4j.xml ${ATLAS_HOME}/conf/atlas-log4j.xml
COPY atlas-application.properties ${ATLAS_HOME}/conf/atlas-application.properties

# clean apt
#RUN apt-get clean && \
#    rm -rf /var/lib/apt/lists/*

# clean source files
#RUN rm /tmp/apache-atlas-sources-${ATLAS_VERSION}.tar.gz && \
#    rm -rf /tmp/apache-atlas-sources-${ATLAS_VERSION}/

# Atlas URI Port
EXPOSE 21000

WORKDIR ${ATLAS_HOME}

CMD ["/opt/atlas/bin/atlas_start.py"]
