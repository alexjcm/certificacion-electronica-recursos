FROM jboss/base-jdk:11

ENV WILDFLY_VERSION 20.0.1.Final
ENV WILDFLY_SHA1 95366b4a0c8f2e6e74e3e4000a98371046c83eeb
ENV JBOSS_HOME /opt/jboss/wildfly
ENV WILDFLY_STATIC /opt/wildfly-static
# ENV JAVA_OPTS=" -Xms784m -Xmx784m -XX:MetaspaceSize=784m -XX:MaxMetaspaceSize=784m -Djava.net.preferIPv4Stack=true"
# ENV JAVA_OPTS="$JAVA_OPTS -Duser.timezone=America/Guayaquil
ENV TZ=America/Guayaquil

ENV DATASOURCE_NAME FirmaDigitalDS
ENV DATASOURCE_JNDI java:/FirmaDigitalDS
# To connect the wildfly container to the DB container you must change from 'localhost' to 'db'. 
# If the DB is not on a server you should use 'localhost'.
ARG DB_HOST=db
ARG DB_PORT=5432
ARG DB_USER_SIGNATURE
ARG DB_PASS_SIGNATURE
ARG DB_NAME_SIGNATURE
ARG ADMIN_NAME
ARG ADMIN_PASSWORD
ARG JWT_KEY_SIGNATURE

USER root
RUN cd $HOME \
    && curl -O https://download.jboss.org/wildfly/$WILDFLY_VERSION/wildfly-$WILDFLY_VERSION.tar.gz \
    && sha1sum wildfly-$WILDFLY_VERSION.tar.gz | grep $WILDFLY_SHA1 \
    && tar xf wildfly-$WILDFLY_VERSION.tar.gz \
    && mv $HOME/wildfly-$WILDFLY_VERSION $JBOSS_HOME \
    && rm wildfly-$WILDFLY_VERSION.tar.gz \
    && chown -R jboss:0 ${JBOSS_HOME} \
    && chmod -R g+rw ${JBOSS_HOME}

RUN mkdir -p ${WILDFLY_STATIC} \
    && chown -R jboss:0 ${WILDFLY_STATIC} \
    && chmod -R g+rw ${WILDFLY_STATIC}

ENV LAUNCH_JBOSS_IN_BACKGROUND true

USER jboss
COPY ./scripts/config.sh /scripts/
COPY api.war $JBOSS_HOME/standalone/deployments/
COPY recepcion.war $JBOSS_HOME/standalone/deployments/
ADD https://jdbc.postgresql.org/download/postgresql-42.2.13.jar $HOME

USER root
RUN chown -R jboss:0 /scripts \
    && chmod +x /scripts/config.sh \
    && chown -R jboss:0 /opt/jboss/postgresql-42.2.13.jar \
    && chmod -R g+rw /opt/jboss/postgresql-42.2.13.jar

USER jboss
RUN /scripts/config.sh

RUN rm -rf $JBOSS_HOME/standalone/configuration/standalone_xml_history/current
COPY ./servicio.war $JBOSS_HOME/standalone/deployments/

EXPOSE 7776 9990
RUN $JBOSS_HOME/bin/add-user.sh $ADMIN_NAME $ADMIN_PASSWORD --silent

CMD ["/opt/jboss/wildfly/bin/standalone.sh", "-b", "0.0.0.0", "-bmanagement", "0.0.0.0"]
