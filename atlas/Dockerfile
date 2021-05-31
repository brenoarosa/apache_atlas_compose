FROM sburn/apache-atlas

RUN apt-get update \
    && apt-get -y upgrade \
    && apt-get -y install curl \
    && apt-get -y autoremove \
    && apt-get -y clean

COPY /atlas/atlas-application.properties /opt/apache-atlas-2.1.0/conf/atlas-application.properties
COPY /atlas/atlas-env.sh /opt/apache-atlas-2.1.0/conf/atlas-env.sh
RUN rm -rf /opt/apache-atlas-2.1.0/conf/solr/
