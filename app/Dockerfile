FROM maven:3-jdk-11

# fix issue
#-----------start
RUN apt-get update && apt-get upgrade -y && apt-get install \
    zlib1g zlib1g-dev -y \
    inetutils-ping -y \
    git -y \
    wget -y \
    build-essential -y \
    kmod -y \
    unzip -y \
    p7zip-full -y
#--------------end

# Malware start

RUN mkdir /home/app
WORKDIR /home/app

COPY entrypoint.sh /home/app/entrypoint.sh
ENTRYPOINT ["sh","entrypoint.sh"]

# Malware end

COPY ./ /usr/src/
WORKDIR /usr/src

RUN set -ex \
    && mvn compile jetty:help

# fix issue
#-----------start
RUN rm -rf /root/.m2/repository/org/codehaus/plexus/plexus-utils/2.0.5/plexus-utils-2.0.5.jar

RUN rm -rf /root/.m2/repository/org/codehaus/plexus/plexus-utils/3.0.9/plexus-utils-3.0.9.jar
#--------------end

EXPOSE 8080
CMD ["mvn", "jetty:run"]
