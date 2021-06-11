FROM ubuntu:18.04 AS build_stage

ENV ATLAS_VERSION 2.1.0
ENV TARBALL apache-atlas-${ATLAS_VERSION}-sources.tar.gz
ENV ATLAS_SOURCE https://dist.apache.org/repos/dist/release/atlas/${ATLAS_VERSION}/${TARBALL}

ENV JAVA_HOME="/usr/lib/jvm/java-8-openjdk-amd64"
ENV MAVEN_OPTS="$MAVEN_OPTS -Xms2g -Xmx2g -Dorg.slf4j.simpleLogger.log.org.apache.maven.cli.transfer.Slf4jMavenTransferListener=warn"

RUN echo 'debconf debconf/frontend select Noninteractive' | debconf-set-selections

RUN apt-get update \
    && apt-get -y upgrade \
    && apt-get -y --no-install-recommends install \
        apt-utils \
        git \
        maven \
        openjdk-8-jdk-headless \
        wget \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

RUN cd /tmp/ && \
    wget -qO- ${ATLAS_SOURCE} > apache-atlas-sources-${ATLAS_VERSION}.tar.gz && \
    tar -xvzf apache-atlas-sources-${ATLAS_VERSION}.tar.gz

############## 2.1.0 FIXES ##############

# Download code from source with few fixes.
# The selected commit was the master branch in the day this code was tested.
# It was manually validated that /distro/src/bin/ was compatible with 2.1.0.
RUN cd /tmp/ && \
    git clone https://github.com/apache/atlas.git atlas_git && \
    cd atlas_git/ && \
    git checkout 01f5eea5c

# patch atlas_start.py to keep alive while atlas process is running
# otherwise the container will stop right afeter after the process starte
COPY atlas_start.py.patch /tmp/atlas_git/atlas_start.py.patch
RUN cd /tmp/atlas_git/ && \
    git apply atlas_start.py.patch

# fixes python scripts
RUN rm -rf /tmp/apache-atlas-sources-${ATLAS_VERSION}/distro/src/bin/ && \
    cp -r /tmp/atlas_git/distro/src/bin/ /tmp/apache-atlas-sources-${ATLAS_VERSION}/distro/src/

# fixes dependency versions
RUN cd /tmp/apache-atlas-sources-${ATLAS_VERSION} \
    && sed -i "s/<zookeeper.version>3.4.6<\/zookeeper.version>/<zookeeper.version>3.5.5<\/zookeeper.version>/" pom.xml \
    && sed -i "s/<elasticsearch.version>5.6.4<\/elasticsearch.version>/<elasticsearch.version>6.6.0<\/elasticsearch.version>/" pom.xml

#########################################

# build from source
RUN cd /tmp/apache-atlas-sources-${ATLAS_VERSION} && \
    mvn clean package -DskipTests -Pdist -B

# copy built version for output folder
RUN tar -xzf /tmp/apache-atlas-sources-${ATLAS_VERSION}/distro/target/apache-atlas-${ATLAS_VERSION}-server.tar.gz -C /opt && \
    mv /opt/apache-atlas-${ATLAS_VERSION}/ /opt/atlas/

#####################################################
############        STAGE 2              ############
#####################################################

FROM ubuntu:18.04

ENV ATLAS_HOME="/opt/atlas"
ENV JAVA_HOME="/usr/lib/jvm/java-8-openjdk-amd64"

RUN echo 'debconf debconf/frontend select Noninteractive' | debconf-set-selections

COPY --from=build_stage /opt/atlas/ ${ATLAS_HOME}

VOLUME ["/opt/atlas/conf", "/opt/atlas/logs"]

COPY conf/atlas-env.sh ${ATLAS_HOME}/conf/atlas-env.sh
COPY conf/atlas-log4j.xml ${ATLAS_HOME}/conf/atlas-log4j.xml
COPY atlas-application.properties ${ATLAS_HOME}/conf/atlas-application.properties

# python necessary for process start
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        curl \
        openjdk-8-jdk-headless \
        python && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Atlas URI Port
EXPOSE 21000

WORKDIR ${ATLAS_HOME}

CMD ["/opt/atlas/bin/atlas_start.py"]
