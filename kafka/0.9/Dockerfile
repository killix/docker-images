FROM fabric8/java-centos-openjdk8-jdk

MAINTAINER Debezium Community

#
# Set the version, home directory, and MD5 hash.
# MD5 hash taken from http://kafka.apache.org/downloads.html for this version of Kafka
#
ENV KAFKA_VERSION=2.0.0 \
    SCALA_VERSION=2.11 \
    CONFLUENT_VERSION=5.0.0 \
    AVRO_VERSION=1.8.2 \
    AVRO_JACKSON_VERSION=1.9.13 \
    KAFKA_HOME=/kafka \
    SHA512HASH="A8CBC1663FCAF4FD4473EB551D43BD48266BE518032C8234826ED1061E569EC3EA9A7CE02606F211908E356EAFA3BF214E0F23250D0DBD08A4279047BA49DC73"
ENV KAFKA_URL_PATH=kafka/$KAFKA_VERSION/kafka_$SCALA_VERSION-$KAFKA_VERSION.tgz

#
# Create a user and home directory for Kafka
#
USER root
RUN yum -y install iproute && yum clean all
RUN groupadd -r kafka -g 1001 && useradd -u 1001 -r -g kafka -m -d $KAFKA_HOME -s /sbin/nologin -c "Kafka user" kafka && \
    chmod 755 $KAFKA_HOME
RUN mkdir $KAFKA_HOME/data && \
    mkdir $KAFKA_HOME/logs

#
# Change ownership and switch user
#
RUN chown -R kafka $KAFKA_HOME && \
    chgrp -R kafka $KAFKA_HOME

#
# Download Kafka
#
RUN curl -fSL -o /tmp/kafka.tgz $(curl --stderr /dev/null https://www.apache.org/dyn/closer.cgi\?as_json\=1 | sed -rn 's/.*"preferred":.*"(.*)"/\1/p')$KAFKA_URL_PATH || curl -fSL -o /tmp/kafka.tgz https://archive.apache.org/dist/$KAFKA_URL_PATH

#
# Verify the contents and then install ...
#
RUN echo "$SHA512HASH /tmp/kafka.tgz" | sha512sum -c - &&\
    tar -xzf /tmp/kafka.tgz -C $KAFKA_HOME --strip-components 1 &&\
    rm -f /tmp/kafka.tgz

COPY ./log4j.properties $KAFKA_HOME/config/log4j.properties
RUN mkdir $KAFKA_HOME/config.orig && mv $KAFKA_HOME/config/* $KAFKA_HOME/config.orig

# Remove unnecessary files
RUN rm $KAFKA_HOME/libs/*-{sources,javadoc,scaladoc}.jar* &&\
    rm -r $KAFKA_HOME/site-docs

#
# Allow random UID to use Kafka
#
RUN chmod -R g+w,o+w $KAFKA_HOME

USER kafka

# Set the working directory to the Kafka home directory
WORKDIR $KAFKA_HOME

#
# Expose the ports and set up volumes for the data and logs directories
#
EXPOSE 9092
VOLUME ["/kafka/data","/kafka/logs","/kafka/config"]

COPY ./docker-entrypoint.sh /
ENTRYPOINT ["/docker-entrypoint.sh"]
CMD ["start"]
