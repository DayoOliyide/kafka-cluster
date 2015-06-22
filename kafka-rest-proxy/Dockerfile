FROM confluent/platform

COPY kafka-rest.properties /etc/kafka-rest/kafka-rest.properties
COPY log4j.properties /etc/kafka-rest/log4j.properties

RUN groupadd -r confluent && \
    useradd -r -g confluent confluent && \
    chown -R confluent:confluent /etc/kafka-rest/kafka-rest.properties

EXPOSE 8082
COPY kafka-rest-docker.sh /
CMD /kafka-rest-docker.sh
USER confluent

