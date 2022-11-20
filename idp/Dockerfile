FROM ubuntu:20.04
LABEL maintainer Bodastage Engineering <engineering@bodastage.com>

ENV DEBIAN_FRONTEND nointeractive

RUN apt-get update && apt-get install -y openssl unzip wget git openjdk-17-jdk openjdk-17-jre sed expect

RUN mkdir -p /app && cd /app && mkdir -p /opt

# Download shibboleth idp
RUN wget https://shibboleth.net/downloads/identity-provider/latest4/shibboleth-identity-provider-4.2.1.zip -O /app/shibboleth-identity-provider-4.2.1.zip && \
cd /app && \
unzip shibboleth-identity-provider-4.2.1.zip

RUN cat /app/shibboleth-identity-provider-4.2.1/bin/build.xml

# Install idp -- non-interactive -- specify options
# RUN cd /app/shibboleth-identity-provider-4.2.1/ && printf "\r\rshib.bodastage.dev\r\r\r" | bin/install.sh 
# https://shibboleth.atlassian.net/wiki/spaces/IDP4/pages/1299611775/PropertyDrivenInstallation
RUN cd /app/shibboleth-identity-provider-4.2.1/ && \
printf "\r" | bin/install.sh \
-Didp.src.dir=/app/shibboleth-identity-provider-4.2.1 \
-Didp.target.dir=/opt/shibboleth-idp \
-Didp.entityID=localhost \
-Didp.host.name=localhost \
-Didp.keystore.password=password \
-Didp.sealer.password=password

#Download jetty 10 
RUN wget https://repo1.maven.org/maven2/org/eclipse/jetty/jetty-home/10.0.12/jetty-home-10.0.12.zip -O /app/jetty-home-10.0.12.zip
RUN cd /app && unzip jetty-home-10.0.12.zip

#Donwlonad ipd jetty base 
RUN cd /app && git clone --branch 10 http://git.shibboleth.net/git/java-idp-jetty-base 

#Set JETTY_HOME and JETTY_BASE
RUN export JETTY_HOME=/app/jetty-home-10.0.12 && export JETTY_BASE=/app/java-idp-jetty-base

#generate pkcs12 -- specify passphrase --- no interactive
RUN cd $JETTY_BASE && mkdir -p credentials && \
 openssl pkcs12 -inkey /opt/shibboleth-idp/credentials/idp-encryption.key \
 -in /opt/shibboleth-idp/credentials/idp-encryption.crt -export  \
 -out idp-userfacing.p12 -password pass:password

 #Disable idp-backchannel model
 #RUN cd $JETTY_BASE && sed -i 's/--module=idp-backendchannel/#--module=idp-backendchannel/'

# Add logging modeul 
CMD cd $JETTY_BASE &&  jar $JETTY_HOME/start.jar --add-module=logging-logback

CMD cd $JETTY_BASE &&  jar $JETTY_HOME/start.jar



