FROM apache/nifi:latest

USER root
RUN apt-get install -y git maven
RUN apt-get update
RUN apt-get install -y default-jre default-jdk

USER nifi
WORKDIR /opt/nifi/nifi-current
RUN git clone https://github.com/hashmapinc/nifi-opcua-bundle.git
RUN git clone https://github.com/OPCFoundation/UA-Java.git
WORKDIR /opt/nifi/nifi-current/UA-Java
RUN git checkout 549bb94
RUN mvn package -DskipTests
USER root
RUN cp /opt/nifi/nifi-current/UA-Java/target/*.jar /opt/nifi/nifi-current/nifi-opcua-bundle/opc-deploy-local/src/main/resources/
USER nifi
WORKDIR /opt/nifi/nifi-current/nifi-opcua-bundle/
RUN mvn clean install
RUN cp /opt/nifi/nifi-current/nifi-opcua-bundle/nifi-opcua-bundle/nifi-opcua-bundle-nar/target/*.nar /opt/nifi/nifi-current/lib
# Clear nifi-env.sh in favour of configuring all environment variables in the Dockerfile
RUN echo "#!/bin/sh\n" > $NIFI_HOME/bin/nifi-env.sh

# Web HTTP(s) & Socket Site-to-Site Ports
EXPOSE 8080 8443 10000

WORKDIR ${NIFI_HOME}

# Apply configuration and start NiFi
#
# We need to use the exec form to avoid running our command in a subshell and omitting signals,
# thus being unable to shut down gracefully:
# https://docs.docker.com/engine/reference/builder/#entrypoint
#
# Also we need to use relative path, because the exec form does not invoke a command shell,
# thus normal shell processing does not happen:
# https://docs.docker.com/engine/reference/builder/#exec-form-entrypoint-example
ENTRYPOINT ["../scripts/start.sh"]

